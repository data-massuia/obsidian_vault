# Fuel iX — My Project: Sales Intelligence Copilot

> **Goal**: Build a Fuel iX Copilot that lets the marketing team query daily sales data using natural language — no SQL, no dashboards, just conversation.  
> **Tags**: #fuelix #project #bigquery #sales #marketing #copilot #architecture

---

## 🎯 Project Summary

| Item | Detail |
|------|--------|
| **Copilot Name** | Sales Intelligence Assistant |
| **Audience** | Marketing team |
| **Data Source** | BigQuery table (order-level daily sales data) |
| **Data Freshness** | Daily summaries (updated each morning) |
| **Integration Method** | BigQuery → Cloud Scheduler → Google Drive → Fuel iX Knowledge Hub |
| **Privacy Approach** | Pre-aggregated summaries only — no raw/PII data exposed |
| **Copilot Type** | Official (org-wide) or Shared (team-only) |

---

## 🏗️ Architecture

### Why This Approach?

Your company has strict data privacy policies, so a **live database connection** (via MCP) is not the right starting point. Instead, we use a **pre-aggregated export pipeline**:

- ✅ No live connection between Fuel iX and BigQuery
- ✅ Only aggregated, anonymized summaries leave your GCP environment
- ✅ Uses Fuel iX's **native Google Drive integration** (no custom code on the Fuel iX side)
- ✅ Security team can review and approve the export pipeline independently
- ✅ You control exactly what data is exposed

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     YOUR GCP ENVIRONMENT                        │
│                                                                 │
│  BigQuery                                                       │
│  (order-level sales data)                                       │
│         │                                                       │
│         ▼  (nightly at ~2 AM)                                   │
│  Cloud Scheduler                                                │
│         │                                                       │
│         ▼                                                       │
│  Cloud Run / Cloud Function                                     │
│  (runs aggregation SQL queries)                                 │
│         │                                                       │
│         ▼  (exports structured summaries)                       │
│  Google Drive Folder                                            │
│  📁 /Sales Intelligence Copilot/                               │
│     ├── daily_sales_summary_YYYY-MM-DD.md                      │
│     ├── weekly_performance_summary.md                          │
│     └── monthly_trends.md                                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ (Google Drive auto-sync every 24h)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     FUEL iX PLATFORM                            │
│                                                                 │
│  Knowledge Hub                                                  │
│  (indexes the Google Drive files via RAG)                       │
│         │                                                       │
│         ▼                                                       │
│  Sales Intelligence Copilot                                     │
│  (system prompt + knowledge base + tools)                       │
│         │                                                       │
│         ▼                                                       │
│  Marketing Team                                                 │
│  "What were our top 5 products last week?"                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Data Pipeline Design

### What to Aggregate from BigQuery

Your BigQuery table has **order-level data** (one row per order). The nightly job should aggregate this into human-readable summaries. Here's what to compute:

#### Daily Summary (generated each morning for yesterday)
```sql
-- Revenue by product/category
SELECT 
  product_name,
  category,
  COUNT(*) AS orders,
  SUM(revenue) AS total_revenue,
  AVG(order_value) AS avg_order_value
FROM orders
WHERE order_date = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)
GROUP BY product_name, category
ORDER BY total_revenue DESC;

-- Revenue by region/channel
SELECT 
  region,
  channel,
  COUNT(*) AS orders,
  SUM(revenue) AS total_revenue
FROM orders
WHERE order_date = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)
GROUP BY region, channel
ORDER BY total_revenue DESC;

-- Day-over-day comparison
SELECT 
  SUM(CASE WHEN order_date = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY) 
      THEN revenue END) AS yesterday_revenue,
  SUM(CASE WHEN order_date = DATE_SUB(CURRENT_DATE(), INTERVAL 2 DAY) 
      THEN revenue END) AS day_before_revenue,
  SUM(CASE WHEN order_date = DATE_SUB(CURRENT_DATE(), INTERVAL 8 DAY) 
      THEN revenue END) AS same_day_last_week_revenue
FROM orders
WHERE order_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 8 DAY);
```

#### Weekly Summary (generated every Monday for last week)
- Revenue by product (top 10 and bottom 10)
- Revenue by region and channel
- Week-over-week % change
- New vs. returning customer orders
- Average order value trend

#### Monthly Summary (generated on the 1st for last month)
- Month-over-month revenue comparison
- Year-over-year comparison
- Top performing products/categories
- Channel mix evolution
- Running MTD and YTD totals

### Output Format for Google Drive

Export as **Markdown files** (`.md`) — they are text-based, well-structured, and RAG indexes them very effectively.

**Example: `daily_sales_summary_2026-03-13.md`**
```markdown
# Daily Sales Summary — March 13, 2026

**Data as of:** March 12, 2026 (yesterday)
**Generated:** March 13, 2026 at 2:00 AM

## Overall Performance
- **Total Revenue:** $142,350
- **Total Orders:** 1,847
- **Average Order Value:** $77.07
- **vs. Previous Day:** +8.3% revenue, +5.1% orders
- **vs. Same Day Last Week:** +12.1% revenue

## Top 5 Products by Revenue
| Product | Orders | Revenue | Avg Order |
|---------|--------|---------|-----------|
| Product A | 234 | $28,450 | $121.58 |
| Product B | 189 | $22,100 | $116.93 |
...

## Revenue by Region
| Region | Orders | Revenue | vs. Yesterday |
|--------|--------|---------|---------------|
| West | 612 | $54,200 | +11.2% |
| East | 498 | $43,100 | +6.8% |
...

## Revenue by Channel
| Channel | Orders | Revenue | Share |
|---------|--------|---------|-------|
| Online | 1,102 | $89,400 | 62.8% |
| In-Store | 745 | $52,950 | 37.2% |
```

---

## 📝 System Prompt for the Copilot

Copy this into the Fuel iX Copilot system prompt field:

```
You are the Sales Intelligence Assistant for the marketing team.

Your role is to help marketing professionals understand our daily sales 
performance using clear, plain language — no SQL, no jargon, just insights.

## What You Know
You have access to daily, weekly, and monthly sales summaries that are 
updated every morning. These summaries include:
- Daily revenue by product, category, region, and sales channel
- Day-over-day, week-over-week, and month-over-month comparisons
- Order volume, average order value, and revenue trends
- Top and bottom performing products

## How to Respond
- Always mention the date range of the data you're referencing
- Present revenue with $ and use commas for thousands (e.g., $142,350)
- Show percentage changes with + or - and one decimal place (e.g., +8.3%)
- Highlight notable trends or anomalies proactively — don't just answer the 
  literal question, add context if something stands out
- If comparing periods, always show both the absolute number AND the % change
- Keep responses concise but complete — use bullet points and tables when helpful
- If you don't have the data to answer a question, say so clearly and suggest 
  what question you CAN answer

## What You Do NOT Do
- You do not have access to individual customer data or any PII
- You do not make predictions or forecasts (only report on actual data)
- You do not answer questions unrelated to sales performance
- You do not make up numbers — only use data from your knowledge base
- If asked about data older than what's in your summaries, acknowledge the limitation

## Data Freshness
Your data is updated every morning. If a user asks about "today," clarify 
that the most recent data available is from yesterday (the previous business day).
```

---

## 💬 Sample Questions the Marketing Team Can Ask

Once the Copilot is live, the marketing team can ask questions like:

**Performance Questions:**
- "What were our total sales yesterday?"
- "Which were our top 5 products last week by revenue?"
- "How did online sales compare to in-store last month?"
- "What's our best performing region this week?"

**Trend Questions:**
- "Is revenue trending up or down compared to last week?"
- "Which product category has grown the most this month?"
- "How does this week compare to the same week last year?"

**Diagnostic Questions:**
- "Which products had the biggest drop in orders yesterday?"
- "What's our average order value trend over the last 30 days?"
- "Which channel has the highest average order value?"

**Summary Questions:**
- "Give me a quick summary of last week's performance"
- "What are the 3 most important things I should know about yesterday's sales?"
- "Prepare a brief for our Monday marketing meeting about last week"

---

## ⏰ Scheduled Tasks to Set Up

Once the Copilot is live, configure these automated tasks:

| Task | Schedule | Delivery |
|------|----------|---------|
| Daily sales brief | Every weekday at 8:00 AM | Email + in-app |
| Weekly performance summary | Every Monday at 8:00 AM | Email + in-app |
| Monthly highlights | 1st of each month at 8:00 AM | Email + in-app |
| Friday wrap-up | Every Friday at 4:00 PM | Email + in-app |

**Example task prompt to give the Copilot:**
> "Every Monday at 8:00 AM, summarize last week's sales performance, highlight the top 3 products, note any significant week-over-week changes, and flag anything unusual."

---

## 🛠️ Implementation Steps

### Phase 1: Data Pipeline (GCP Side) — ~1-2 weeks

- [ ] **Design the aggregation queries** — finalize what metrics to include in each summary
- [ ] **Build the Cloud Function** — Python script that runs the BigQuery queries and formats output as Markdown
- [ ] **Set up Cloud Scheduler** — trigger the function nightly at 2 AM
- [ ] **Create the Google Drive folder** — set up `/Sales Intelligence Copilot/` with appropriate sharing permissions
- [ ] **Test the pipeline** — run manually, verify output quality and formatting
- [ ] **Get security approval** — have your security team review the pipeline (data stays in GCP, only aggregated summaries go to Google Drive)

### Phase 2: Fuel iX Copilot Setup — ~1-2 days

- [ ] **Log into app.fuelix.ai**
- [ ] **Create a new Custom Copilot** named "Sales Intelligence Assistant"
- [ ] **Paste the system prompt** (from above)
- [ ] **Select the LLM model** — recommend **Claude Sonnet 4** (best at reasoning over structured data, 1M token context)
- [ ] **Enable tools**: Date & Time, Reasoning, Scheduled Tasks
- [ ] **Connect Google Drive** in the Knowledge Hub → point to your `/Sales Intelligence Copilot/` folder
- [ ] **Wait for initial sync** (up to 24 hours for first index)

### Phase 3: Testing — ~1 week

- [ ] **Test with 20+ realistic questions** from the list above
- [ ] **Identify gaps** — what questions can't it answer? Add those metrics to the pipeline
- [ ] **Refine the system prompt** based on response quality
- [ ] **Invite 2-3 marketing team members** for beta testing
- [ ] **Collect feedback** and iterate

### Phase 4: Launch

- [ ] **Share with the full marketing team** (Shared Copilot)
- [ ] **Ask admin to promote to Official Copilot** for org-wide visibility
- [ ] **Set up Scheduled Tasks** for automated daily/weekly briefs
- [ ] **Create a quick-start guide** for the marketing team (what questions to ask, what data is available)

---

## 🔮 Phase 2: Live BigQuery Connection (Future)

Once you've proven the value and built trust with your security team, consider upgrading to a **live MCP connection**:

**What changes:**
- Instead of daily summaries, the Copilot queries BigQuery in real-time
- Marketing team can ask about data from any time period, any granularity
- No 24-hour lag — data is always current

**What's needed:**
- A **BigQuery MCP Server** (open-source options exist on GitHub, or build custom)
- Security team approval for the network connection
- GCP service account with read-only BigQuery access
- Registration of the MCP server with Fuel iX's MCP Registry

**Security controls you can add:**
- Row-level security in BigQuery (restrict what data the service account can see)
- Query guardrails in the MCP server (block certain table/column access)
- IP allowlisting (only allow connections from Fuel iX's IP range)
- Audit logging of all queries

---

## 📋 Key Decisions & Open Questions

| Question | Status | Notes |
|----------|--------|-------|
| Do we have Google Workspace? | ✅ Yes | Enables native Google Drive sync |
| Security approval for Google Drive export? | ⬜ Pending | Need to confirm aggregated data is OK |
| Which BigQuery table(s) to use? | ⬜ TBD | Need to identify the right BI layer table |
| What columns/metrics are available? | ⬜ TBD | Need to review schema |
| Who owns the Google Drive folder? | ⬜ TBD | Should be a service account or shared drive |
| Fuel iX admin access? | ⬜ TBD | Need to confirm who can create Official Copilots |
| Phase 2 MCP timeline? | ⬜ Future | Revisit after Phase 1 proves value |

---

## 📚 Related Notes
- [[Fuel iX — Platform Overview]]
- [[Fuel iX — How to Create a Copilot]]
