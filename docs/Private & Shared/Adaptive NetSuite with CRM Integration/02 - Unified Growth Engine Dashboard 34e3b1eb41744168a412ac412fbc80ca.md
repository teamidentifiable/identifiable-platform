# 02 - Unified Growth Engine Dashboard

# Unified Growth Engine Dashboard

The "Growth Engine" dashboard replaces NetSuite's traditional "Home" screen with a unified view that integrates ERP (Revenue), CRM (Pipeline), and Marketing (Lead Source) data into a single, actionable interface.

---

## Dashboard Purpose & Audience

**Primary Users:**

- **Executives:** Need high-level visibility into ARR, pipeline health, and cash runway
- **Sales Leaders:** Monitor weighted pipeline, deal velocity, and customer health
- **Marketing Leaders:** Track attribution, ROAS, and campaign performance
- **Finance Leaders:** View burn rate, runway, and revenue recognition timing

**Key Insight:** This dashboard eliminates the need to open separate systems (NetSuite for financials, Salesforce for pipeline, HubSpot for marketing) by presenting a unified view of business health.

---

## Mock Data Structure

The dashboard uses a unified JSON data structure that represents the output of the GraphQL Mesh middleware. This data structure serves as the single source of truth for all dashboard components.

### Complete Data Schema

```json
{
  "financials": {
    "arr": 4520000,
    "arr_growth_yoy": 0.18,
    "burn_rate": 125000,
    "cash_runway_months": 22,
    "cash_on_hand": 2750000,
    "monthly_recurring_revenue": 376666,
    "gross_margin": 0.72
  },
  "crm_pipeline": [
    {
      "stage": "Discovery",
      "value": 1200000,
      "count": 15,
      "win_prob": 0.2,
      "avg_deal_size": 80000,
      "avg_days_in_stage": 18
    },
    {
      "stage": "Proposal",
      "value": 850000,
      "count": 8,
      "win_prob": 0.5,
      "avg_deal_size": 106250,
      "avg_days_in_stage": 25
    },
    {
      "stage": "Negotiation",
      "value": 420000,
      "count": 3,
      "win_prob": 0.8,
      "avg_deal_size": 140000,
      "avg_days_in_stage": 12
    },
    {
      "stage": "Closed Won (This Q)",
      "value": 980000,
      "count": 12,
      "win_prob": 1.0,
      "avg_deal_size": 81666,
      "avg_days_in_stage": 0
    }
  ],
  "marketing_attribution": [
    {
      "campaign": "Q1_Webinar_Series",
      "spend": 15000,
      "attributed_revenue": 145000,
      "roas": 9.6,
      "leads_generated": 340,
      "sql_conversion_rate": 0.18,
      "avg_deal_size": 95000
    },
    {
      "campaign": "LinkedIn_Retargeting",
      "spend": 8500,
      "attributed_revenue": 32000,
      "roas": 3.7,
      "leads_generated": 125,
      "sql_conversion_rate": 0.12,
      "avg_deal_size": 85000
    },
    {
      "campaign": "Organic_Search",
      "spend": 2000,
      "attributed_revenue": 210000,
      "roas": 105.0,
      "leads_generated": 580,
      "sql_conversion_rate": 0.22,
      "avg_deal_size": 120000
    }
  ],
  "customer_health": {
    "avg_nps": 52,
    "nps_trend": 0.08,
    "churn_risk_accounts": 5,
    "churn_risk_arr": 340000,
    "upsell_opportunities": 12,
    "upsell_potential_arr": 890000,
    "avg_health_score": 78,
    "accounts_with_low_usage": 8
  }
}
```

### Data Source Mapping

| Data Object | Source System | API Endpoint | Refresh Frequency |
| --- | --- | --- | --- |
| `financials.arr` | NetSuite | `/record/v1/consolidatedBalance` | Real-time (WebSocket) |
| `financials.burn_rate` | NetSuite | `/record/v1/cashFlow` | Daily at 2am |
| `crm_pipeline` | Salesforce | `/services/data/v57.0/query` (SOQL) | Every 5 minutes |
| `marketing_attribution` | Marketo/HubSpot | `/rest/v1/leads/programs` | Hourly |
| `customer_health.avg_nps` | Delighted/Wootric | `/v1/metrics` | Daily at 3am |
| `customer_health.churn_risk_accounts` | Predictive Model | Internal API `/ml/churn_score` | Every 15 minutes |

---

## UI Layout: The "Growth Engine" View

The dashboard uses a **12-column grid system** for flexible, responsive layouts. The layout is organized into three horizontal rows (A, B, C) that stack vertically.

### Row A: The Pulse (Metrics Bar)

**Purpose:** Provide at-a-glance visibility into the four most critical business metrics.

**Layout:** Full-width (12 columns), height 100px

**Visual Design:**

- White background with subtle bottom border
- Four equal-width cards (3 columns each)
- Each card contains: metric value (large), metric label (small), trend indicator (icon + percentage), and sparkline background

#### Metric 1: Global ARR

**Display:**

```
$4.52M
↑ 18% YoY
Global ARR
[Sparkline graph showing 12-month trend]
```

**Calculation Logic:**

```jsx
const arr = financials.arr;
const growthRate = financials.arr_growth_yoy;
const trendDirection = growthRate >= 0 ? '↑' : '↓';
const trendColor = growthRate >= 0.15 ? 'green' : growthRate >= 0 ? 'yellow' : 'red';
```

**Sparkline Data Source:**

- Query: NetSuite ARR for past 12 months (monthly snapshots)
- Visualization: Line chart, 200px wide × 30px tall, rendered in background

#### Metric 2: Weighted Pipeline

**Display:**

```
$1.34M
Weighted Pipeline
[Progress bar showing pipeline coverage ratio]
```

**Calculation Logic:**

```jsx
const weightedPipeline = crm_pipeline.reduce((sum, stage) => {
  return sum + (stage.value * stage.win_prob);
}, 0);

// Pipeline Coverage = Weighted Pipeline / Quarterly Target
const quarterlyTarget = financials.arr * 0.25; // Assuming even quarterly distribution
const coverageRatio = weightedPipeline / quarterlyTarget;
```

**Color Coding:**

- Green: Coverage ratio ≥ 3.0 (healthy pipeline)
- Yellow: Coverage ratio 2.0-2.9 (adequate pipeline)
- Red: Coverage ratio < 2.0 (insufficient pipeline)

#### Metric 3: Marketing ROAS

**Display:**

```
8.2x
Marketing ROAS
[Donut chart showing channel contribution]
```

**Calculation Logic:**

```jsx
const totalSpend = marketing_attribution.reduce((sum, c) => sum + c.spend, 0);
const totalRevenue = marketing_attribution.reduce((sum, c) => sum + c.attributed_revenue, 0);
const roas = totalRevenue / totalSpend;
```

**Integration Note:**

- Real-time integration with Google Ads API for paid spend
- Real-time integration with LinkedIn Campaign Manager for social spend
- Attribution model: Multi-touch (weighted)

#### Metric 4: Cash Runway

**Display:**

```
22 Months
Cash Runway
[Burn rate indicator]
```

**Calculation Logic:**

```jsx
const runway = financials.cash_on_hand / financials.burn_rate;
const runwayMonths = Math.floor(runway);
```

**Data Source:**

- Live bank feed sync via Plaid or similar banking API
- Updated every transaction (real-time)

---

### Row B: The Adaptive Intelligence Layer

**Layout:** Split into two sections

- Left: 8 columns (Revenue Velocity Chart)
- Right: 4 columns (Smart Actions Feed)

#### Left Block: Revenue Velocity Chart

**Purpose:** Visualize the relationship between marketing spend, pipeline creation, and revenue recognition over time.

**Chart Type:** Stacked bar chart with overlaid line graph

**Time Period:** Last 6 months (monthly aggregation)

**Data Series:**

1. **Marketing Spend (Red Bars)** - Bottom layer
    - Source: `marketing_attribution.spend` aggregated by month
    - Y-axis: Dollars spent
2. **Pipeline Created (Blue Bars)** - Middle layer
    - Source: Salesforce opportunities created, grouped by month
    - Y-axis: Total pipeline value
3. **Recognized Revenue (Green Bars)** - Top layer
    - Source: NetSuite revenue recognition schedule
    - Y-axis: Revenue recognized in accounting period
4. **Conversion Rate Line (Purple)** - Overlaid
    - Calculation: (Recognized Revenue / Pipeline Created) × 100
    - Y-axis (right): Percentage

**Interactive Features:**

**Hover Tooltip Example:**

```
December 2024
Marketing Spend: $15,000 ↑ 32% vs. Nov
Pipeline Created: $145,000 (Jan '25)
Revenue Recognized: $89,000 (Feb '25)

Insight: 45-day lag from spend to revenue
```

**Business Insight:**

This chart shows the **lag time between marketing spend and ERP revenue recognition**. For example:

- Dec '24: Spend spiked $15k (Webinar campaign)
- Jan '25: Pipeline spiked $145k (direct attribution)
- Feb '25: Revenue recognized $89k (61% close rate)

#### Right Block: Smart Actions Feed

**Purpose:** Surface AI-driven insights and provide one-click action buttons to resolve issues.

**Layout:** Vertical stack of action cards

**Card Structure:**

- Icon (left, color-coded by urgency)
- Alert text (main content)
- Action button (right, primary CTA)

#### Action Card Example 1: Churn Risk

```
🔴 5 Churn Risk Accounts Detected

Low product usage logs combined with overdue invoices indicate high churn probability.

Affected ARR: $340,000

[Enroll in Retention Sequence] (Button)
```

**Data Sources:**

- Product usage logs from analytics platform
- Invoice status from NetSuite
- NPS scores from customer success platform
- ML churn prediction model

**Action Trigger:**

Clicking "Enroll in Retention Sequence" triggers:

1. HubSpot workflow: Sends personalized outreach email from CSM
2. Salesforce task: Creates follow-up task for account manager
3. Slack notification: Alerts customer success team channel

#### Action Card Example 2: Inventory Alert

```
🟡 Inventory Low: MacBook Pro M3

Current stock: 7 units
Avg monthly sales: 12 units
Runway: 17 days

[Create PO #9921] (Button)
```

**Data Sources:**

- Current inventory levels from NetSuite
- Sales velocity calculation (rolling 90-day average)
- Supplier lead times from vendor management system

**Action Trigger:**

Clicking "Create PO #9921" triggers:

1. NetSuite: Creates draft purchase order with pre-filled details
2. Opens PO in slide-out editor for review and approval
3. Auto-calculates optimal order quantity based on EOQ model

---

### Row C: Deep Dive Tables (Retool Style)

**Layout:** Full-width (12 columns), variable height based on row count

**Purpose:** Provide a unified view of active deals combined with their financial status—data that traditionally requires toggling between CRM and ERP systems.

#### Table: Active Deals + Invoice Status (Combined View)

**Innovation:** This table merges CRM pipeline data (Salesforce) with ERP financial data (NetSuite) in a single view, eliminating the need to cross-reference multiple systems.

**Column Structure:**

| Column Name | Width | Data Source | Data Type | Purpose |
| --- | --- | --- | --- | --- |
| Client Name | 15% | Salesforce Account | Text | Primary identifier |
| CRM Stage | 12% | Salesforce Opportunity | Select | Sales pipeline position |
| Probability | 10% | Calculated | Progress Bar | Win likelihood |
| Last Marketing Touch | 18% | Marketo/HubSpot | Text + Icon | Attribution context |
| Open Invoices (ERP) | 12% | NetSuite | Currency | Outstanding AR |
| Payment Status | 13% | NetSuite | Status Badge | Collection risk |
| Last Login | 10% | Product Analytics | Date | Engagement indicator |
| Actions | 10% | N/A | Button | Quick actions menu |

**Sample Data (Rendered):**

```
┌────────────┬─────────────┬─────────────┬───────────────────────┬──────────────┬─────────────────┬────────────┐
│ Client     │ CRM Stage   │ Probability │ Last Marketing Touch  │ Open Balance │ Payment Status  │ Last Login │
├────────────┼─────────────┼─────────────┼───────────────────────┼──────────────┼─────────────────┼────────────┤
│ Acme Corp  │ Negotiation │ 80% [████] │ CEO Podcast (3d ago)  │ $12,450.00   │ ⚠️ Overdue (15d) │ Today      │
│ Stark Ind  │ Proposal    │ 50% [██  ] │ Whitepaper Download   │ $0.00        │ N/A             │ 2d ago     │
│ Wayne Ent  │ Closed Won  │ 100%[████] │ Direct Referral       │ $55,000.00   │ ✅ Paid          │ Today      │
│ Cyberdyne  │ Discovery   │ 20% [█   ] │ LinkedIn Ad           │ $0.00        │ No History      │ Never      │
└────────────┴─────────────┴─────────────┴───────────────────────┴──────────────┴─────────────────┴────────────┘
```

#### Adaptive Filter Feature

**Smart Filter: "Overdue Invoices"**

When a user filters the table to show only rows with overdue invoices, the system automatically:

1. **Highlights the "Last Marketing Touch" column** to surface whether marketing campaigns are targeting customers who owe money
2. **Adds a "Win-back Campaign" badge** next to customers enrolled in re-engagement flows
3. **Dims the "CRM Stage" column** since these customers shouldn't be pursued for new deals until AR is resolved

**Business Impact:**

This prevents sales teams from closing new deals with customers who have outstanding payments, reducing bad debt and improving cash flow.

#### Table Styling Specifications

**Header Row:**

- Background: `#F9FAFB` (Light gray)
- Text: `#6B7280` (Medium gray)
- Font: Inter Medium, 11px, uppercase, tracked (0.05em)
- Height: 44px
- Sticky positioning (remains visible during scroll)

**Data Rows:**

- Background: `#FFFFFF` (White)
- Hover state: `#F3F4F6` (Light gray)
- Selected state: `#EEF2FF` (Light indigo)
- Border: `1px solid #E5E7EB` between rows
- Height: 56px (adequate touch target)

**Status Badges:**

- Paid: Green pill with checkmark icon
- Overdue: Red/orange pill with alert icon, includes days overdue
- N/A: Gray pill, no icon

---

## Business Logic & Calculations

### Weighted Pipeline Calculation

```jsx
function calculateWeightedPipeline(pipeline) {
  return pipeline.reduce((total, stage) => {
    // Multiply stage value by win probability
    const weighted = stage.value * stage.win_prob;
    return total + weighted;
  }, 0);
}

// Example:
// Discovery: $1,200,000 × 0.2 = $240,000
// Proposal: $850,000 × 0.5 = $425,000
// Negotiation: $420,000 × 0.8 = $336,000
// Total Weighted Pipeline: $1,001,000
```

### Churn Risk Detection Algorithm

```jsx
function detectChurnRisk(account) {
  let riskScore = 0;
  
  // Factor 1: NPS Score (40% weight)
  if (account.nps < 30) riskScore += 40;
  else if (account.nps < 50) riskScore += 20;
  
  // Factor 2: Product Usage (30% weight)
  const usageTrend = account.last_30d_logins / account.previous_30d_logins;
  if (usageTrend < 0.5) riskScore += 30;
  else if (usageTrend < 0.8) riskScore += 15;
  
  // Factor 3: Payment Status (30% weight)
  if (account.days_overdue > 30) riskScore += 30;
  else if (account.days_overdue > 0) riskScore += 15;
  
  // Classification
  if (riskScore >= 60) return 'HIGH';
  if (riskScore >= 40) return 'MEDIUM';
  return 'LOW';
}
```

---

## Performance Optimization

### Data Loading Strategy

**Initial Page Load:**

1. Fetch Row A metrics (priority 1, < 500ms target)
2. Fetch Row B chart data (priority 2, < 1s target)
3. Fetch Row C table data (priority 3, < 2s target)

**Caching Strategy:**

- Row A: Cache for 30 seconds (refresh every 30s)
- Row B: Cache for 5 minutes (refresh every 5m)
- Row C: Cache for 2 minutes (refresh every 2m)

**Pagination:**

- Table shows 10 rows initially
- "Load More" button fetches next 10 rows
- Infinite scroll option available in user preferences

---

## Next Steps

With the dashboard structure and data model defined, proceed to:

- **Subpage 03:** Data Architecture & Integration Layer (GraphQL implementation)
- **Subpage 04:** Growth Table Component (Full React code)