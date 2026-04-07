# BigQuery — TV Telemetry Query Review & Diagnostics

**Date:** 2026-04-07
**Topic:** BigQuery SQL review, JOIN diagnostics, telemetry vs customer ref matching

---

## Table of Contents
1. [[#Original Query Overview]]
2. [[#Query Review — What's Working Well]]
3. [[#Query Review — Potential Issues]]
4. [[#Understanding QUALIFY ROW_NUMBER]]
5. [[#Diagnosing the 47k Unmatched Devices]]
6. [[#Diagnostic Queries]]
7. [[#Root Causes Found]]
8. [[#Customer ID Comparison Queries]]
9. [[#Commscope CTE Replacement]]
10. [[#Final Recommendations]]

---

## Original Query Overview

The query pulls TV box telemetry data from `bq_telemetry` (CommScope/Opus platform), calculates hours each device was actively watching TV (STB on, TV on), then joins to `bq_tv_customer_ref` to enrich with customer demographics. The final output is a summary grouped by customer type, combo type, and account classification.

**Tables used:**
- `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry` — raw device telemetry events
- `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref` — customer reference/demographics

**CTE Pipeline:**
1. `deduplicated_source` — deduplicates telemetry by `(dt, serial_num)`, keeping latest `audit.created_ts`
2. `parsed_telemetry` — unnests the `event` array, extracts standby status, HDMI status, and uptime (capped at 905s)
3. `Telemetry_Usage` — aggregates hours per device per month (`Stb_On_Tv_On`)
4. `customer_classification` — pulls customer demographics, deduplicates by device per month
5. Final SELECT — joins telemetry to customer ref, groups by customer type/combo/classification

---

## Query Review — What's Working Well

1. **Deduplication in Step 1** — `QUALIFY ROW_NUMBER() OVER (PARTITION BY dt, serial_num ORDER BY audit.created_ts DESC) = 1` cleanly keeps the latest record per device per day.
2. **`LEAST(e.usp_agent.delta_uptime, 905)`** — Smart cap to avoid inflated uptime values from missed events.
3. **`QUALIFY` in `customer_classification`** — Correctly deduplicates by device per month using the latest update timestamp.
4. **`LEFT JOIN`** — Correct choice; keeps all telemetry devices even if they don't have a matching customer record.
5. **`COUNT(DISTINCT cust.Account_ID)`** — Good way to count unique accounts linked to devices.

---

## Query Review — Potential Issues

| Issue | Severity |
|---|---|
| `UNNEST` drops devices with empty/null event arrays | ⚠️ Medium — confirm intent |
| `tv_pltfrm_typ_cd` selected in CTE but unused in final output | ⚠️ Low — possible oversight |
| Commented filter `cust_pltfrm_typ_cd` has wrong column name (should be `tv_pltfrm_typ_cd`) | ⚠️ Low — will error if uncommented |
| `LENGTH(serial_num) > 0` doesn't explicitly handle NULLs (works but less readable) | ℹ️ Cosmetic |
| `LOWER(is_standby)` is redundant — BOOLEAN cast to STRING is already lowercase | ℹ️ Cosmetic |

**Details:**

- **`UNNEST` behavior:** If `t.event` is an empty array `[]` or `NULL`, the cross-join `UNNEST` produces zero rows for that device — effectively dropping it. Use `LEFT JOIN UNNEST(t.event) AS e` if you want to keep devices with no events.
- **Devices with no customer match:** With a `LEFT JOIN`, unmatched devices still appear in the final output with `NULL` customer fields — all grouped into a single `NULL` row in the `GROUP BY`. Add `WHERE cust.Device_ID IS NOT NULL` if you only want matched devices.

---

## Understanding QUALIFY ROW_NUMBER

```sql
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY Report_Month, TRIM(CAST(tv_dvc_extrnl_id AS STRING))
    ORDER BY tv_dvc_src_updt_ts DESC
) = 1
```

**Step by step:**

1. **`PARTITION BY Report_Month, Device_ID`** — Groups rows into independent buckets per device per month. Each device gets its own bucket; `ROW_NUMBER()` restarts at 1 for each.
2. **`ORDER BY tv_dvc_src_updt_ts DESC`** — Within each bucket, sorts rows from most recently updated to oldest.
3. **`ROW_NUMBER() = 1`** — Keeps only the most recently updated row per device per month.
4. **`QUALIFY`** — BigQuery's way of filtering on window functions (like a `WHERE` clause that runs after window functions are calculated).

**Key insight:** `PARTITION BY` creates separate buckets per device — so if a customer has devices ABC123, ABC124, ABC125, each gets its own bucket and all three survive. Only duplicate rows for the *same* device in the *same* month are deduplicated.

**Example:**

| Report_Month | Device_ID | tv_dvc_src_updt_ts | ROW_NUMBER |
|---|---|---|---|
| 2026-03-01 | ABC123 | 2026-03-15 10:00 | 1 ✅ kept |
| 2026-03-01 | ABC123 | 2026-03-10 08:00 | 2 ❌ dropped |
| 2026-03-01 | ABC124 | 2026-03-12 09:00 | 1 ✅ kept |
| 2026-03-01 | ABC125 | 2026-03-08 11:00 | 1 ✅ kept |

---

## Diagnosing the 47k Unmatched Devices

The main query showed ~47,037 devices in a `NULL` row (no customer match). The device IDs looked like `SER254311000003813`, `SEI...`, etc. A VLOOKUP in Google Sheets was able to match many of them, suggesting the issue was in the SQL logic, not missing data.

**Investigation steps taken:**

1. **Side-by-side bracketed comparison** — Confirmed `exact_match = TRUE` for a known device. Values were identical, same length, no hidden characters.
2. **Non-ASCII character detection** — `dirty_count = 0` on both sides. No invisible characters.
3. **Raw join (no CAST/TRIM)** — Still 40,595 unmatched. Confirmed data truly doesn't exist in `tv_customer_ref` for those devices.
4. **Removed `tv_acct_extrnl_id IS NOT NULL` filter** — Gap closed from 47k → 40k. Confirmed ~7k devices exist in the ref table but have no account ID linked.
5. **Prefix/length breakdown** — Remaining 40k devices are genuinely not present in `tv_customer_ref`.

---

## Diagnostic Queries

### 1. Side-by-Side Bracketed Comparison
Confirms whether the raw values from both tables are truly identical for a known device:

```sql
DECLARE start_date DATE DEFAULT DATE_TRUNC(DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH), MONTH);
DECLARE end_date   DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

WITH telemetry_sample AS (
    SELECT DISTINCT
        serial_num AS telemetry_id,
        CONCAT('[', serial_num, ']') AS telemetry_bracketed,
        LENGTH(serial_num) AS telemetry_len
    FROM `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry`
    WHERE dt >= start_date AND dt < end_date
      AND serial_num LIKE 'SER254311000003813%'
    LIMIT 1
),
ref_sample AS (
    SELECT DISTINCT
        CAST(tv_dvc_extrnl_id AS STRING) AS ref_id,
        CONCAT('[', CAST(tv_dvc_extrnl_id AS STRING), ']') AS ref_bracketed,
        LENGTH(CAST(tv_dvc_extrnl_id AS STRING)) AS ref_len
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND CAST(tv_dvc_extrnl_id AS STRING) LIKE 'SER254311000003813%'
    LIMIT 1
)
SELECT
    t.telemetry_id, t.telemetry_bracketed, t.telemetry_len,
    r.ref_id, r.ref_bracketed, r.ref_len,
    t.telemetry_id = r.ref_id AS exact_match
FROM telemetry_sample t
CROSS JOIN ref_sample r
```

### 2. Non-ASCII Dirty Record Detection

```sql
DECLARE start_date DATE DEFAULT DATE_TRUNC(DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH), MONTH);
DECLARE end_date   DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

WITH telemetry_check AS (
    SELECT serial_num AS original_id,
        REGEXP_REPLACE(serial_num, r'[^\x20-\x7E]', '?') AS cleaned_id,
        serial_num = REGEXP_REPLACE(serial_num, r'[^\x20-\x7E]', '') AS is_clean
    FROM `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry`
    WHERE dt >= start_date AND dt < end_date AND serial_num LIKE 'SER%'
),
ref_check AS (
    SELECT CAST(tv_dvc_extrnl_id AS STRING) AS original_id,
        REGEXP_REPLACE(CAST(tv_dvc_extrnl_id AS STRING), r'[^\x20-\x7E]', '?') AS cleaned_id,
        CAST(tv_dvc_extrnl_id AS STRING) = REGEXP_REPLACE(CAST(tv_dvc_extrnl_id AS STRING), r'[^\x20-\x7E]', '') AS is_clean
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND CAST(tv_dvc_extrnl_id AS STRING) LIKE 'SER%'
)
SELECT 'telemetry' AS source, COUNTIF(NOT is_clean) AS dirty_count, COUNT(*) AS total FROM telemetry_check
UNION ALL
SELECT 'customer_ref' AS source, COUNTIF(NOT is_clean) AS dirty_count, COUNT(*) AS total FROM ref_check
```

### 3. Raw Match Count (no CAST/TRIM)

```sql
DECLARE start_date DATE DEFAULT DATE_TRUNC(DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH), MONTH);
DECLARE end_date   DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

SELECT
    COUNT(DISTINCT t.serial_num) AS total_telemetry_devices,
    COUNT(DISTINCT CASE WHEN r.tv_dvc_extrnl_id IS NOT NULL THEN t.serial_num END) AS matched_devices,
    COUNT(DISTINCT CASE WHEN r.tv_dvc_extrnl_id IS NULL THEN t.serial_num END) AS unmatched_devices,
    (
        SELECT COUNT(DISTINCT tv_dvc_extrnl_id)
        FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
        WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
          AND LOWER(tv_dvc_extrnl_id) LIKE '%ser%'
    ) AS total_ref_ser_devices
FROM (
    SELECT DISTINCT serial_num
    FROM `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry`
    WHERE dt >= start_date AND dt < end_date AND LENGTH(serial_num) > 0
) t
LEFT JOIN (
    SELECT DISTINCT tv_dvc_extrnl_id
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND tv_dvc_extrnl_id IS NOT NULL
) r ON t.serial_num = r.tv_dvc_extrnl_id
```

### 4. Unmatched Device Diagnosis (with categorization)

```sql
-- (uses full CTE pipeline — see Root Causes section)
-- Final SELECT categorizes each unmatched device:
SELECT
    u.Device_ID, u.Report_Month, u.Stb_On_Tv_On,
    r.tv_acct_extrnl_id AS Account_ID_in_ref,
    CASE
        WHEN r.tv_dvc_extrnl_id IS NULL THEN 'Not in customer_ref at all'
        WHEN r.tv_acct_extrnl_id IS NULL THEN 'In customer_ref but no Account ID'
        ELSE 'Has Account ID (unexpected)'
    END AS diagnosis
FROM unmatched_devices u
LEFT JOIN (
    SELECT DISTINCT TRIM(CAST(tv_dvc_extrnl_id AS STRING)) AS Device_ID,
        tv_dvc_extrnl_id, tv_acct_extrnl_id
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND tv_dvc_extrnl_id IS NOT NULL
) r ON u.Device_ID = r.Device_ID
ORDER BY diagnosis, u.Device_ID
```

### 5. Devices in customer_ref with no Account ID (truly missing)

```sql
DECLARE start_date DATE DEFAULT DATE_TRUNC(DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH), MONTH);
DECLARE end_date   DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

WITH device_account_check AS (
    SELECT
        TRIM(CAST(tv_dvc_extrnl_id AS STRING)) AS Device_ID,
        COUNTIF(tv_acct_extrnl_id IS NOT NULL) AS rows_with_account,
        COUNTIF(tv_acct_extrnl_id IS NULL) AS rows_without_account
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND tv_dvc_extrnl_id IS NOT NULL
      AND LOWER(tv_clnt_typ_cd) LIKE 'telustv%'
    GROUP BY 1
)
SELECT Device_ID, rows_with_account, rows_without_account
FROM device_account_check
WHERE rows_with_account = 0  -- truly never has an account ID
ORDER BY Device_ID
```

---

## Root Causes Found

Two distinct root causes were identified for the 47k unmatched devices:

### Root Cause 1: `tv_acct_extrnl_id IS NOT NULL` filter dropping ~7k devices
The `customer_classification` CTE had `AND tv_acct_extrnl_id IS NOT NULL`. Some devices exist in `tv_customer_ref` but have no account ID linked (the field is `NULL`). This filter silently excluded them from the CTE, causing the join to fail.

**Fix:** Remove `AND tv_acct_extrnl_id IS NOT NULL` from the `customer_classification` CTE. The account ID is only needed for counting in the final SELECT — it doesn't need to be a join prerequisite.

**Result:** Gap closed from 47k → 40k unmatched devices.

### Root Cause 2: ~40k devices genuinely not in `tv_customer_ref`
After all query logic fixes, 40,595 devices from `bq_telemetry` have no corresponding record in `tv_customer_ref` at all — confirmed by a raw join with no CAST/TRIM. These devices are sending telemetry but are not registered in the customer reference table.

**Next step:** Escalate to data engineering team with a list of the unmatched device IDs for investigation.

> **Note on `tv_customer_ref` table behavior:** This table is a daily snapshot — it adds new rows every day but never deletes old ones. Every device that has ever been provisioned should appear in it. Devices missing from this table are a genuine data gap.

---

## Customer ID Comparison Queries

### Count matched vs unmatched customers (telemetry vs customer ref)

```sql
DECLARE start_date DATE DEFAULT DATE_TRUNC(DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH), MONTH);
DECLARE end_date   DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

SELECT
    COUNT(DISTINCT t.customer_id) AS total_telemetry_customers,
    COUNT(DISTINCT CASE WHEN r.tv_acct_extrnl_id IS NOT NULL THEN t.customer_id END) AS matched_customers,
    COUNT(DISTINCT CASE WHEN r.tv_acct_extrnl_id IS NULL THEN t.customer_id END) AS unmatched_customers,
    (
        SELECT COUNT(DISTINCT tv_acct_extrnl_id)
        FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
        WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
          AND LOWER(tv_clnt_typ_cd) LIKE 'telustv%'
          AND tv_acct_extrnl_id IS NOT NULL
    ) AS total_ref_customers
FROM (
    SELECT DISTINCT account[SAFE_OFFSET(0)].user_name AS customer_id
    FROM `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry`
    WHERE dt >= start_date AND dt < end_date
      AND account[SAFE_OFFSET(0)].user_name IS NOT NULL
      AND LENGTH(account[SAFE_OFFSET(0)].user_name) > 0
) t
LEFT JOIN (
    SELECT DISTINCT TRIM(CAST(tv_acct_extrnl_id AS STRING)) AS tv_acct_extrnl_id
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND LOWER(tv_clnt_typ_cd) LIKE 'telustv%'
      AND tv_acct_extrnl_id IS NOT NULL
) r ON t.customer_id = r.tv_acct_extrnl_id
```

> **Key field mapping:**
> - Telemetry customer ID: `account[SAFE_OFFSET(0)].user_name`
> - Customer ref account ID: `tv_acct_extrnl_id`

### List of unmatched Customer IDs

```sql
DECLARE start_date DATE DEFAULT DATE_TRUNC(DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH), MONTH);
DECLARE end_date   DATE DEFAULT DATE_TRUNC(CURRENT_DATE(), MONTH);

WITH
deduplicated_source AS (
    SELECT *
    FROM `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry`
    WHERE dt >= start_date AND dt < end_date
    QUALIFY ROW_NUMBER() OVER (PARTITION BY dt, serial_num ORDER BY audit.created_ts DESC) = 1
),
telemetry_customers AS (
    SELECT DISTINCT
        DATE_TRUNC(dt, MONTH) AS Report_Month,
        account[SAFE_OFFSET(0)].user_name AS Customer_ID
    FROM deduplicated_source
    WHERE account[SAFE_OFFSET(0)].user_name IS NOT NULL
      AND LENGTH(account[SAFE_OFFSET(0)].user_name) > 0
),
customer_classification AS (
    SELECT DISTINCT
        DATE(DATE_TRUNC(tv_cust_ref_dt, MONTH)) AS Report_Month,
        TRIM(CAST(tv_acct_extrnl_id AS STRING)) AS Account_ID
    FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref`
    WHERE tv_cust_ref_dt >= start_date AND tv_cust_ref_dt < end_date
      AND tv_acct_extrnl_id IS NOT NULL
)
SELECT DISTINCT t.Customer_ID, t.Report_Month
FROM telemetry_customers t
LEFT JOIN customer_classification cust
    ON t.Customer_ID = cust.Account_ID
    AND t.Report_Month = cust.Report_Month
WHERE cust.Account_ID IS NULL
ORDER BY t.Customer_ID
```

---

## Commscope CTE Replacement

The old `commscope` CTE pulled from a staging table (`bi-stg-cabe-pr-dfe151.tableau_dataset.bq_commscope_device`) with a static date filter. It was replaced with the full telemetry pipeline from `bq_telemetry`.

**Use case:** Compare devices in `bq_tv_program_watch_view` / `bq_tv_stb_vod_playback_view` against telemetry to check sync.

```sql
-- 1. Deduplicate telemetry source
deduplicated_source AS (
    SELECT *
    FROM `bi-srv-tv-opus-pr-3b70af.commscope_v2.bq_telemetry`
    WHERE dt >= start_date AND dt < end_date
    QUALIFY ROW_NUMBER() OVER (PARTITION BY dt, serial_num ORDER BY audit.created_ts DESC) = 1
),
-- 2. Parse events
parsed_telemetry AS (
    SELECT
        t.serial_num AS Device_ID,
        CAST(e.is_standby AS STRING) AS is_standby,
        e.hdmi.status AS hdmi_status,
        LEAST(e.usp_agent.delta_uptime, 905) AS duration_seconds
    FROM deduplicated_source t,
    UNNEST(t.event) AS e
    WHERE LENGTH(serial_num) > 0
),
-- 3. Calculate hours per device (replaces old commscope CTE)
commscope AS (
    SELECT
        Device_ID AS Stb_Id,
        CAST(ROUND(SUM(CASE WHEN LOWER(is_standby) = 'false' AND LOWER(hdmi_status) = 'enabled'
            THEN duration_seconds ELSE 0 END) / 3600, 2) AS NUMERIC) AS Stb_On_Tv_On
    FROM parsed_telemetry
    GROUP BY 1
)
```

> **Red flags noted in the sync-check query:**
> - **One-directional comparison** — original query only finds devices in viewership NOT in telemetry. Use `FULL OUTER JOIN` for a complete picture.
> - **Mismatched filters** — viewership tables filter on `West`, `Opus`, `TELUSTV%` but telemetry CTE has no such filters. Not apples-to-apples.
> - **Unnecessary hours calculation** — if only checking device existence, simplify `commscope` to `SELECT DISTINCT serial_num AS Stb_Id` to save cost.

---

## Final Recommendations

### Query Fixes Applied
1. ✅ **Remove `AND tv_acct_extrnl_id IS NOT NULL`** from `customer_classification` CTE — this was silently dropping ~7k devices that exist in the ref table but have no account linked.
2. ✅ **Fix commented-out filter** — `cust_pltfrm_typ_cd` → `tv_pltfrm_typ_cd` if ever uncommented.
3. ⚠️ **Confirm `UNNEST` behavior** — decide if devices with empty/null event arrays should be kept or dropped.

### Data Engineering Escalation
- ~40k devices from `bq_telemetry` have no record in `bq_tv_customer_ref` at all.
- Use the unmatched device list query (Diagnostic Query #4) to export the device IDs and open a bug ticket.

### Sync Check Best Practices
When comparing devices across tables:
- Use `FULL OUTER JOIN` instead of `LEFT JOIN` to catch mismatches in both directions.
- Ensure filters are consistent across all tables being compared.
- Use `COUNTIF(condition)` grouped by device to check if a device *ever* has a value, not just in one row.
- Use `CONCAT('[', value, ']')` to visually expose hidden characters when debugging join mismatches.
- Use `REGEXP_REPLACE(value, r'[^\x20-\x7E]', '?')` to detect non-ASCII/invisible characters.

### Key BigQuery Concepts Used
| Concept | Usage |
|---|---|
| `QUALIFY` | Filter on window functions without a subquery |
| `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)` | Deduplicate keeping latest record per group |
| `UNNEST()` | Flatten repeated/array fields into rows |
| `LEAST()` | Cap a value at a maximum |
| `SAFE_OFFSET(0)` | Access first element of an array without error on empty arrays |
| `COUNTIF()` | Count rows matching a condition (BigQuery-specific) |
| Scalar subquery | Independent `SELECT` inside a `SELECT` returning a single value |
| `FULL OUTER JOIN` | Keep all rows from both tables, matching where possible |
