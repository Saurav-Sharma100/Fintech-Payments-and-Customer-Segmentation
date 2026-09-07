# 💳 Fintech Payments and Customer Segmentation

## 📌 Project Overview

PayFlow is a payment processing platform serving **5,000 merchant customers** across e-commerce, SaaS, and marketplace businesses.

The company processes hundreds of thousands of transactions, but management faces three important questions:

> **Why is the payment failure rate significantly above the industry benchmark?**  
> **Which merchants are showing signs of churn risk?**  
> **How can customer behavior and value be segmented to support better retention decisions?**

This project builds an end-to-end analytics solution using **Google BigQuery and Power BI** to transform raw payment, customer, support, and dispute data into a dimensional model designed for business intelligence.

The project combines three major analytical components:

- **BigQuery SQL** for exploration, transformation, RFM scoring, customer health analysis, and dimensional modelling
- An adapted **RFM framework** for fintech customer segmentation
- A **Star Schema** that integrates customer behavior directly into the analytical model used by Power BI

---

# 📊 Dashboard Preview

<img width="1334" height="744" alt="Image" src="https://github.com/user-attachments/assets/6a853eed-c774-4d49-a16c-da00c4f8f536" />

---

# 📋 Project Snapshot

| Category | Details |
|---|---|
| **Domain** | Fintech / Payment Processing |
| **Business Use Case** | Payment Performance & Customer Segmentation |
| **Customers** | 5,000 merchants |
| **Transactions** | 416,747 |
| **Support Tickets** | ~6,800 |
| **Disputes** | ~3,100 |
| **Data Warehouse** | Google BigQuery |
| **Data Modelling** | Star Schema |
| **Customer Framework** | RFM Segmentation |
| **BI Tool** | Power BI |
| **Primary Fact Table** | `fact_transactions` |
| **Core Dimensions** | Customer, Date, Payment Method, Exchange Rate |
| **Dashboard Focus** | Payment Performance, Customer Health & RFM Segmentation |

---

# 🎯 Business Problem

PayFlow's transaction data showed a payment failure rate of approximately **5%**, compared with an industry benchmark of roughly **2–3%**.

At the same time, the company needed a better way to identify merchants whose declining payment activity, repeated transaction failures, support issues, or disputes could indicate churn risk.

The CEO's three questions were:

1. **Why is the payment failure rate high?**
2. **Which customers are at risk of churning?**
3. **Are different customer segments contributing value differently?**

The objective was therefore not simply to report transactions, but to connect **payment behavior, customer activity and customer health** in a model that could support both executive monitoring and customer retention decisions.

The intended users include:

- Executive leadership
- Operations teams
- Customer Success teams

---

# 🗂️ Data Sources

The project uses four operational datasets:

| Dataset | Business Information |
|---|---|
| **Customers** | Merchant profile, industry, signup date, plan, MRR, country and employee size |
| **Transactions** | Payment amount, status, currency, payment method and processing fee |
| **Support Tickets** | Ticket category, priority, resolution time and satisfaction |
| **Disputes** | Chargebacks, dispute reasons, amounts and outcomes |

The data contains approximately:

- **5,000 customers**
- **416K+ transactions**
- **6.8K support tickets**
- **3.1K disputes**

---

# ☁️ BigQuery Analytics Architecture

BigQuery is used as the primary SQL transformation and analytics environment.

The project separates data into three logical layers:

```text
payflow_raw
     │
     ▼
payflow_analytics
     │
     ▼
payflow_star
     │
     ▼
Power BI
```

### `payflow_raw`

Stores the original operational tables:

```text
customers
transactions
support_tickets
disputes
```

### `payflow_analytics`

Contains customer-level analytical tables derived through SQL:

```text
customer_rfm
customer_segments
customer_health
```

### `payflow_star`

Contains the dimensional model prepared for Power BI reporting.

This separation keeps raw transactional data, analytical calculations and final reporting structures logically distinct.

---

# 🔍 Exploratory Analysis in BigQuery

Before building the RFM framework, SQL was used to understand the major operational patterns in PayFlow's data.

### Transaction Status Distribution

The analysis showed:

- **91.7% successful transactions**
- approximately **5.1% failed transactions**

The failure level is materially higher than the 2–3% benchmark used in the business problem.

### Payment Failures by Method

Failure rates varied by payment method.

The analysis identified **card payments as the highest-failure method**, while digital wallets performed better.

The finished Power BI dashboard preserves this comparison through the **Failure by Method** visual.

### Support Ticket Patterns

Payment failure was also the largest support-ticket category, with **2,322 tickets** and comparatively low satisfaction.

This connected payment reliability with customer-service pressure and supported the need for a broader customer health framework.

---

# 🔄 RFM Framework for Fintech

Traditional RFM analysis is normally built around purchasing behavior:

- **Recency** — how recently a customer purchased
- **Frequency** — how often they purchased
- **Monetary** — how much they spent

For PayFlow, this framework was adapted to fit a **payment processing platform**.

| RFM Component | PayFlow Definition |
|---|---|
| **Recency** | Days since the merchant's last successful transaction |
| **Frequency** | Average successful transactions per month |
| **Monetary** | Average monthly transaction volume processed |

The reasoning is straightforward:

> A merchant who continues processing successful transactions is actively using the platform. A merchant whose transaction activity declines or stops may be showing early signs of churn.

---

# 🧮 RFM Calculation in BigQuery

RFM metrics were calculated at the customer level.

### Recency

```sql
DATE_DIFF(
    CURRENT_DATE(),
    MAX(
        CASE 
            WHEN t.status = 'successful'
            THEN DATE(t.transaction_date)
        END
    ),
    DAY
) AS days_since_last_txn
```

### Frequency

```sql
COUNT(
    CASE 
        WHEN t.status = 'successful'
        THEN t.transaction_id
    END
) /
GREATEST(
    DATE_DIFF(CURRENT_DATE(), c.signup_date, DAY) / 30.0,
    1
) AS avg_monthly_txns
```

### Monetary

```sql
SUM(
    CASE
        WHEN t.status = 'successful'
        THEN t.amount
        ELSE 0
    END
) /
GREATEST(
    DATE_DIFF(CURRENT_DATE(), c.signup_date, DAY) / 30.0,
    1
) AS avg_monthly_volume
```

Each metric is then converted into a score from **1 to 5**, where a higher score represents stronger customer activity or value.

---

# 📊 RFM Scoring

| Score | Recency | Frequency | Monetary |
|---|---|---|---|
| **5** | ≤ 7 days | ≥ 100 / month | ≥ $10K / month |
| **4** | 8–30 days | 50–99 / month | $5K–$10K |
| **3** | 31–60 days | 20–49 / month | $2K–$5K |
| **2** | 61–90 days | 5–19 / month | $500–$2K |
| **1** | > 90 days | < 5 / month | < $500 |

These scores convert different customer behaviors into a consistent structure that can be used for segmentation.

---

# 👥 Customer Segmentation

The RFM scores are combined into behavioral customer segments.

The model includes:

| Segment | Business Meaning |
|---|---|
| **Champions** | Active, frequent and high-value customers |
| **Loyal** | Strong recency and transaction frequency |
| **Whales** | High-value customers with distinctive transaction patterns |
| **At Risk** | Customers showing weakening engagement |
| **Dormant** | Customers with no recent activity |
| **New / Testing** | Recently active customers with limited usage |
| **Needs Attention** | Customers not fitting the main behavioral groups |

The segmentation transforms individual RFM values into categories that can be understood and acted on by business teams.

---

# ❤️ Customer Health Score

RFM is extended with operational risk indicators to create a broader customer health score.

The normalized scoring logic combines:

```text
RFM Base Score
+
Payment Failure Penalty
+
Support Ticket Penalty
+
Dispute Penalty
```

The implemented base score is:

```text
(R × 20) + (F × 20) + (M × 20) - 100
```

Additional penalties are applied when:

- Payment failure rate exceeds **10%** → **-20**
- More than **5 recent support tickets** → **-15**
- More than **2 disputes** → **-15**

Customers are then classified into health categories:

| Health Status | Score |
|---|---:|
| **Healthy** | 70+ |
| **At Risk** | 40–69 |
| **Critical** | Below 40 |

This combines customer engagement with operational friction rather than relying on transaction volume alone.

---

# ⭐ Star Schema Data Model

The dimensional model is one of the central components of the project.

### Power BI Model Screenshot

<img width="1077" height="736" alt="Image" src="https://github.com/user-attachments/assets/c5b92ed3-c77c-4d8e-bba1-31b807b1d9f4" />

---

The model is centered around `fact_transactions`, with dimensions used to filter and analyze transactional activity.

```text
                   dim_date
                       │
                       │
dim_customer ── fact_transactions ── dim_payment_method
                       │
                       │
               dim_exchange_rate
```

---

# 🧱 Fact Table — `fact_transactions`

The fact table stores the individual payment events.

Its grain is:

> **One row per transaction**

Key fields include:

- `transaction_id`
- `date_key`
- `customer_key`
- `payment_method_key`
- `transaction_date`
- `status`
- `currency`
- `transaction_amount`
- `transaction_fee`

The table also includes pre-calculated analytical flags:

```sql
CASE WHEN status = 'failed'
     THEN 1 ELSE 0
END AS is_failed
```

```sql
CASE WHEN status = 'successful'
     THEN 1 ELSE 0
END AS is_successful
```

```sql
CASE WHEN status = 'refunded'
     THEN 1 ELSE 0
END AS is_refunded
```

```sql
CASE WHEN status = 'disputed'
     THEN 1 ELSE 0
END AS is_disputed
```

These simplify downstream aggregation in both SQL and Power BI.

---

# 👤 `dim_customer` — RFM Integrated into the Dimension

The most important modelling decision was to embed the RFM analysis directly into `dim_customer`.

The dimension contains:

### Customer Attributes

- Business name
- Industry
- Plan
- MRR
- Country
- Employee size
- Signup date

### RFM Metrics

- Days since last transaction
- Average monthly transactions
- Average monthly volume

### RFM Scores

- Recency Score
- Frequency Score
- Monetary Score

### Customer Intelligence

- Segment
- Payment failure rate
- Recent support tickets
- Total disputes
- Health score
- Health status

Embedding these fields directly into the customer dimension means that customer segment and health information can filter transactional measures throughout the Power BI model without requiring a separate RFM table relationship.

---

# 📅 Supporting Dimensions

### `dim_date`

Provides reusable calendar attributes including:

- Date
- Year
- Month
- Month name
- Day
- Day of week
- Quarter
- Weekend indicator

### `dim_payment_method`

Standardizes payment methods into user-friendly reporting values such as:

- Credit / Debit Card
- Bank Transfer
- Digital Wallet

---

# 💱 Multi-Currency Handling

The actual Power BI semantic model also includes a **`dim_exchange_rate`** table.

Because transactions are recorded in multiple currencies, a composite relationship key is created using:

```text
date_key + currency
```

This links each transaction to the appropriate exchange rate for that date and currency.

The model then calculates standardized USD values such as:

```dax
transaction_amount_usd =
fact_transactions[transaction_amount]
    * RELATED(dim_exchange_rate[to_usd_rate])
```

```dax
successful_amount_usd =
fact_transactions[successful_amount]
    * RELATED(dim_exchange_rate[to_usd_rate])
```

This allows transaction performance to be compared using a common currency.

---

# 🔗 Star Schema Relationships

The primary relationships in the finished model are:

```text
fact_transactions[customer_key]
        → dim_customer[customer_key]

fact_transactions[date_key]
        → dim_date[date_key]

fact_transactions[payment_method_key]
        → dim_payment_method[payment_method_key]

fact_transactions[fact_key]
        → dim_exchange_rate[fact_key]
```

The result is a model where the transaction fact table can be filtered consistently by customer behavior, date, payment type and exchange rate.

This is what makes the RFM framework operationally useful inside Power BI rather than leaving it as an isolated SQL analysis.

---

# 📐 DAX Measures

The DAX shown below comes from the finished Power BI semantic model.

### Total Volume

```dax
Total Volume =
SUM(fact_transactions[successful_amount_usd])
```

The final dashboard therefore reports successful transaction volume after currency conversion into USD.

### Success Rate

```dax
Success Rate =
DIVIDE(
    SUM(fact_transactions[is_successful]),
    COUNT(fact_transactions[transaction_id]),
    0
)
```

### Failure Rate

```dax
Failure Rate =
DIVIDE(
    SUM(fact_transactions[is_failed]),
    COUNT(fact_transactions[transaction_id]),
    0
)
```

### Average Health Score

```dax
Avg Health Score =
AVERAGE(dim_customer[Health_Score])
```

### At Risk MRR

```dax
At Risk MRR =
CALCULATE(
    SUM(dim_customer[mrr]),
    dim_customer[Segment] = "At Risk"
)
```

These measures connect the transaction fact table and RFM-enriched customer dimension to the executive dashboard.

---

# 📈 Power BI Dashboard

The finished Power BI report presents the main business indicators on a single executive dashboard.

### KPI Cards

The dashboard reports:

| KPI | Result |
|---|---:|
| **Total Volume** | $19.07M |
| **Success Rate** | 91.7% |
| **Average Health Score** | 48.9 |
| **At Risk MRR** | $4.1K |

---

## Monthly Revenue Trend

Tracks successful transaction volume across time to show changes in payment activity.

Because total volume is calculated from the USD-converted transaction amount, different transaction currencies can be compared on the same scale.

---

## Status Distribution

Shows the breakdown between:

- Successful
- Failed
- Refunded
- Disputed transactions

The dashboard shows approximately **91.75% successful transactions** and approximately **5.08% failed transactions**, reinforcing the payment reliability issue identified during exploration.

---

## Failure by Method

Compares failure rates across:

- Credit / Debit Card
- Bank Transfer
- Digital Wallet

Card payments show the highest failure rate, while digital wallets show the lowest of the three methods.

This gives operations teams a clear view of where payment reliability problems are concentrated.

---

## RFM Segmentation

The RFM scatter plot visualizes the customer segments using the RFM scoring framework.

It allows the dashboard user to compare customer groups based on behavioral activity rather than looking only at revenue or transaction counts.

The report also includes a detailed customer-level table containing RFM scores, MRR, health score, failure indicators and support-ticket information.

This makes it possible to move from an executive metric directly to the merchants contributing to that result.

---

# 🛠️ Tools & Technologies

| Tool | Use |
|---|---|
| **Google BigQuery** | Data storage, SQL analysis, RFM calculations and star-schema transformation |
| **SQL** | EDA, customer segmentation, health scoring and dimensional modelling |
| **Power BI** | Data modelling, DAX and executive dashboard |
| **DAX** | Transaction KPIs, customer health measures and USD-based reporting |
| **Google Sheets / GOOGLEFINANCE** | Time-anchored currency exchange-rate preparation |
| **RFM Framework** | Merchant behavioral segmentation |
| **Star Schema** | Analytics-optimized dimensional model |

---

# 💡 Business Insights

The completed analysis highlights several important patterns.

### 1. Payment failures remain a material operational issue

The dashboard reports a success rate of approximately **91.7%**, leaving failures materially above the benchmark referenced in the business problem.

### 2. Failure risk differs by payment method

Credit and debit card transactions have the highest failure rate among the payment methods analyzed, while digital wallets perform better.

This provides a specific operational area for further investigation.

### 3. Customer risk cannot be understood from transaction value alone

RFM adds behavioral context by measuring how recently merchants have transacted, how frequently they process payments and how much volume they generate.

The health score extends this further by incorporating payment failures, support activity and disputes.

### 4. The customer dimension connects behavior directly to transaction performance

Embedding customer segments and health information into `dim_customer` allows the same customer classification to be used directly against transaction measures throughout the Power BI model.

This connects customer retention analysis with payment performance rather than treating them as separate reporting problems.

---

# 📦 Project Deliverables

The project produces:

- BigQuery raw-data layer
- BigQuery analytical RFM tables
- Customer segmentation table
- Customer health table
- Star-schema dimensional model
- Currency exchange-rate dimension
- Power BI semantic model
- DAX measures
- Executive Power BI dashboard
- RFM customer segmentation analysis

---

# 🏁 Conclusion

This project was built to answer a broader fintech question than simply **how many payments succeeded or failed**.

Starting with customer, transaction, support and dispute data, BigQuery was used to explore payment performance, calculate an adapted fintech RFM framework, classify merchants into behavioral segments and create customer health indicators.

The key architectural step was then integrating that customer intelligence directly into a **star schema**, with `fact_transactions` at the center and customer, date, payment-method and exchange-rate dimensions supporting analysis.

The finished Power BI model connects those layers into one reporting experience: payment performance can be analyzed alongside merchant behavior, customer health and churn risk.

The result is an analytics solution that moves from **raw payment activity → customer intelligence → dimensional modelling → executive decision support**.
