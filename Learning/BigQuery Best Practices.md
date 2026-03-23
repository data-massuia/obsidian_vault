# BigQuery Best Practices

*Learned from real query debugging sessions — March 2026*

---

## Month-End Date Generation

Use `LAST_DAY()` with `GENERATE_DATE_ARRAY` — it's cleaner and handles February/leap years automatically:

```sql
WITH month_end_dates AS (
  SELECT LAST_DAY(month_start) AS month_end_dt
  FROM UNNEST(GENERATE_DATE_ARRAY('2025-01-01', CURRENT_DATE(), INTERVAL 1 MONTH)) AS month_start
  WHERE LAST_DAY(month_start) < DATE_TRUNC(CURRENT_DATE(), MONTH)  -- exclude current incomplete month
)
```

---

## Partition Elimination (Required for Partitioned Tables)

BigQuery requires a **static/literal** filter on the partition column — dynamic values from a joined CTE won't work.

✅ **Correct pattern:** Use `BETWEEN` (static literals) + `IN (SELECT ...)` to narrow to specific dates:

```sql
WHERE tv_cust_profiling_ref_dt BETWEEN '2025-01-31' AND DATE_SUB(DATE_TRUNC(CURRENT_DATE(), MONTH), INTERVAL 1 DAY)
  AND tv_cust_profiling_ref_dt IN (SELECT month_end_dt FROM month_end_dates)
```

❌ **Wrong:** Joining the months CTE directly in the WHERE clause — BigQuery can't use it for partition elimination.

---

## Preventing Fan-Out in JOINs

When joining two tables where one may have duplicates, **aggregate BEFORE joining** to avoid fan-out:

```sql
-- Step 1: Aggregate bq_tv_customer_ref first (one row per account per month)
Account_Box_Flags AS (
  SELECT
    tv_cust_ref_dt AS month_end_dt,
    TRIM(LOWER(tv_acct_extrnl_id)) AS tv_acct_extrnl_id,
    MAX(CASE WHEN ... THEN 1 ELSE 0 END) AS has_4k,
    ...
  FROM `...bq_tv_customer_ref`
  INNER JOIN month_end_dates ON tv_cust_ref_dt = month_end_dates.month_end_dt
  WHERE ...
  GROUP BY tv_cust_ref_dt, TRIM(LOWER(tv_acct_extrnl_id))
),

-- Step 2: Then join to Provisioned_Accounts AFTER aggregation
Filtered_Flags AS (
  SELECT abf.*
  FROM Account_Box_Flags abf
  INNER JOIN Provisioned_Accounts pa
    ON abf.tv_acct_extrnl_id = pa.UHH
    AND abf.month_end_dt = pa.month_end_dt
)
```

---

## Always Use TRIM(LOWER()) on Join Keys

Prevents mismatches due to case differences or leading/trailing whitespace:

```sql
-- In Provisioned_Accounts:
TRIM(LOWER(tv_user_nm)) AS UHH

-- In the JOIN:
ON TRIM(LOWER(ct.tv_acct_extrnl_id)) = pa.UHH
```

---

## Use COUNT(DISTINCT) as a Safety Net

Even after `GROUP BY`, use `COUNT(DISTINCT)` in the final SELECT to guard against any duplicates slipping through:

```sql
SELECT
  month_end_dt AS report_month,
  COUNT(DISTINCT CASE WHEN has_4k = 1 AND has_hd = 0 THEN tv_acct_extrnl_id END) AS UHH_4K_Only,
  COUNT(DISTINCT CASE WHEN NOT (has_4k = 1 AND has_hd = 0) THEN tv_acct_extrnl_id END) AS UHH_Everything_Else,
  COUNT(DISTINCT tv_acct_extrnl_id) AS UHH_Total
FROM Filtered_Flags
GROUP BY month_end_dt
ORDER BY month_end_dt
```

---

## Use DISTINCT in Provisioned_Accounts

Always deduplicate the provisioned accounts list — an account can appear multiple times in the profiling table (e.g. multiple `tv_combo_typ_nm` values for the same date):

```sql
Provisioned_Accounts AS (
  SELECT DISTINCT
    tv_cust_profiling_ref_dt AS month_end_dt,
    TRIM(LOWER(tv_user_nm)) AS UHH
  FROM `...bq_tv_customer_profiling_ref`
  WHERE ...
)
```

---

## Full Template: Monthly Batch Query (West MR)

```sql
WITH month_end_dates AS (
  SELECT LAST_DAY(month_start) AS month_end_dt
  FROM UNNEST(GENERATE_DATE_ARRAY('2025-01-01', CURRENT_DATE(), INTERVAL 1 MONTH)) AS month_start
  WHERE LAST_DAY(month_start) < DATE_TRUNC(CURRENT_DATE(), MONTH)
),

Provisioned_Accounts AS (
  SELECT DISTINCT
    tv_cust_profiling_ref_dt AS month_end_dt,
    TRIM(LOWER(tv_user_nm)) AS UHH
  FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_profiling_ref`
  WHERE
    tv_cust_profiling_ref_dt BETWEEN '2025-01-31' AND DATE_SUB(DATE_TRUNC(CURRENT_DATE(), MONTH), INTERVAL 1 DAY)
    AND tv_cust_profiling_ref_dt IN (SELECT month_end_dt FROM month_end_dates)
    AND tv_combo_typ_nm IN ('budget', 'optik legacy', 'premium')
    AND tv_prpty_txt = 'OPTIK'
    AND tv_stat_cd = 'Active'
    AND tv_acct_classn_txt = 'West'
    AND cust_pltfrm_typ_cd IN ('MR Only', 'Mixed')
),

Account_Box_Flags AS (
  SELECT
    ct.tv_cust_ref_dt AS month_end_dt,
    TRIM(LOWER(ct.tv_acct_extrnl_id)) AS tv_acct_extrnl_id,
    MAX(CASE WHEN tv_clnt_typ_cd IN (
      'UIW4001','UIW4001_2','UIW8001','VIP5602W','VIP5602WT','VIP5662W') THEN 1 ELSE 0 END) AS has_4k,
    MAX(CASE WHEN tv_clnt_typ_cd IN (
      'CIS330','CIS330-GN1-B','CIS430','CIS430-GN1-B','IPN330HD','IPN330HD-SG-A','IPN430MC',
      'IPN4320','IPN4320-SG-A','IPV5050','IPV6015','IPV6016','ISB2200','ISB7000','ISB7050',
      'ISB7100','ISB7105','ISB7150') THEN 1 ELSE 0 END) AS has_hd,
    MAX(CASE WHEN tv_clnt_typ_cd IS NULL THEN 1 ELSE 0 END) AS has_null
  FROM `cio-datahub-enterprise-pr-183a.ent_usage_rated_tv.bq_tv_customer_ref` ct
  INNER JOIN Provisioned_Accounts pa
    ON TRIM(LOWER(ct.tv_acct_extrnl_id)) = pa.UHH
    AND ct.tv_cust_ref_dt = pa.month_end_dt
  WHERE
    ct.tv_cust_ref_dt BETWEEN '2025-01-31' AND DATE_SUB(DATE_TRUNC(CURRENT_DATE(), MONTH), INTERVAL 1 DAY)
    AND ct.tv_cust_ref_dt IN (SELECT month_end_dt FROM month_end_dates)
    AND ct.tv_pltfrm_typ_cd = 'MediaRoom'
    AND ct.tv_cust_typ_cd = 'Optik'
    AND ct.tv_combo_typ_nm != 'pik'
    AND ct.tv_acct_classn_txt = 'West'
  GROUP BY ct.tv_cust_ref_dt, TRIM(LOWER(ct.tv_acct_extrnl_id))
)

SELECT
  month_end_dt                                                                           AS report_month,
  COUNT(DISTINCT CASE WHEN has_4k = 1 AND has_hd = 0 THEN tv_acct_extrnl_id END)       AS UHH_4K_Only,
  COUNT(DISTINCT CASE WHEN NOT (has_4k = 1 AND has_hd = 0) THEN tv_acct_extrnl_id END) AS UHH_Everything_Else,
  COUNT(DISTINCT tv_acct_extrnl_id)                                                     AS UHH_Total
FROM Account_Box_Flags
GROUP BY month_end_dt
ORDER BY month_end_dt
```
