# 04 - Growth Table Component (React)

# Growth Table Component (React)

This component implements the unified "Active Pipeline & Financial Health" table that merges CRM data (Salesforce), ERP data (NetSuite), and Marketing data (Marketo/HubSpot) into a single, high-density Retool-style data grid.

---

## Component Overview

**Purpose:** Display active sales opportunities alongside their financial status, eliminating the need to toggle between separate CRM and ERP systems.

**Key Innovation:** The table shows CRM pipeline stages (Sales context) directly next to overdue invoices (Finance context), helping teams identify risks like negotiating with customers who haven't paid existing bills.

**Technical Stack:**

- React 18+
- TailwindCSS 3.x for styling
- Lucide React for icons
- GraphQL for data fetching (via Apollo Client or React Query)

---

## Prerequisites

**Installation:**

```bash
npm install react lucide-react
# TailwindCSS should already be configured in your project
```

**TailwindCSS Configuration:**

Ensure your `tailwind.config.js` includes these color extensions:

```jsx
module.exports = {
  theme: {
    extend: {
      colors: {
        indigo: {
          50: '#eef2ff',
          600: '#4f46e5',
          700: '#4338ca',
        },
      },
    },
  },
};
```

---

## Complete Component Code

```jsx
import React, { useState } from 'react';
import { MoreHorizontal, ArrowUpRight, AlertCircle, CheckCircle, Clock } from 'lucide-react';

// ============================================================================
// MOCK DATA
// In production, this would come from GraphQL query to unified API
// ============================================================================

const unifiedData = [
  {
    id: 1,
    client: 'Acme Corp',
    contact: 'Road Runner',
    crm_stage: 'Negotiation',
    probability: 80,
    marketing_touch: 'CEO Podcast (3d ago)',
    erp_balance: 12450.00,
    erp_status: 'Overdue',
    erp_days_overdue: 15,
    last_login: 'Today',
  },
  {
    id: 2,
    client: 'Stark Industries',
    contact: 'Pepper Potts',
    crm_stage: 'Proposal',
    probability: 50,
    marketing_touch: 'Whitepaper DL',
    erp_balance: 0.00,
    erp_status: 'N/A',
    erp_days_overdue: 0,
    last_login: '2d ago',
  },
  {
    id: 3,
    client: 'Wayne Enterprises',
    contact: 'Lucius Fox',
    crm_stage: 'Closed Won',
    probability: 100,
    marketing_touch: 'Direct Referral',
    erp_balance: 55000.00,
    erp_status: 'Paid',
    erp_days_overdue: 0,
    last_login: 'Today',
  },
  {
    id: 4,
    client: 'Cyberdyne Systems',
    contact: 'Miles Dyson',
    crm_stage: 'Discovery',
    probability: 20,
    marketing_touch: 'LinkedIn Ad',
    erp_balance: 0.00,
    erp_status: 'No History',
    erp_days_overdue: 0,
    last_login: 'Never',
  },
];

// ============================================================================
// SUB-COMPONENTS
// ============================================================================

/**
 * StatusPill Component
 * Displays payment status with appropriate color coding and icon
 */
const StatusPill = ({ status, days }) => {
  if (status === 'Paid') {
    return (
      <span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-green-100 text-green-800 border border-green-200">
        <CheckCircle className="w-3 h-3 mr-1" /> Paid
      </span>
    );
  }
  if (status === 'Overdue') {
    return (
      <span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-red-100 text-red-800 border border-red-200">
        <AlertCircle className="w-3 h-3 mr-1" /> Overdue ({days}d)
      </span>
    );
  }
  return (
    <span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-gray-100 text-gray-600 border border-gray-200">
      {status}
    </span>
  );
};

/**
 * ProbabilityBar Component
 * Visual indicator of deal win probability
 */
const ProbabilityBar = ({ percent }) => (
  <div className="w-full bg-gray-200 rounded-full h-1.5 mt-2">
    <div
      className={`h-1.5 rounded-full ${percent >= 80 ? 'bg-indigo-600' : 'bg-blue-400'}`}
      style={{ width: `${percent}%` }}
    ></div>
  </div>
);

// ============================================================================
// MAIN COMPONENT
// ============================================================================

export default function GrowthTable() {
  const [selectedRow, setSelectedRow] = useState(null);

  return (
    <div className="p-6 bg-slate-50 min-h-screen font-sans">
      {/* Retool-style Header Block */}
      <div className="flex justify-between items-center mb-4">
        <div>
          <h2 className="text-lg font-bold text-slate-800">Active Pipeline & Financial Health</h2>
          <p className="text-sm text-slate-500">Unified View: Salesforce (CRM) + NetSuite (ERP)</p>
        </div>
        <div className="flex space-x-2">
          <button className="px-3 py-1.5 text-sm bg-white border border-slate-300 text-slate-700 rounded hover:bg-slate-50 shadow-sm transition">
            Filter View
          </button>
          <button className="px-3 py-1.5 text-sm bg-indigo-600 text-white rounded hover:bg-indigo-700 shadow-sm transition">
            + Create Invoice
          </button>
        </div>
      </div>

      {/* The Data Grid */}
      <div className="bg-white border border-slate-200 rounded-lg shadow-sm overflow-hidden">
        <table className="min-w-full divide-y divide-slate-200">
          <thead className="bg-slate-50">
            <tr>
              <th scope="col" className="px-6 py-3 text-left text-xs font-semibold text-slate-500 uppercase tracking-wider">
                Client / Contact
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-semibold text-slate-500 uppercase tracking-wider">
                CRM Stage
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-semibold text-slate-500 uppercase tracking-wider">
                Mktg Context
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-semibold text-slate-500 uppercase tracking-wider">
                Open Balance (ERP)
              </th>
              <th scope="col" className="px-6 py-3 text-right text-xs font-semibold text-slate-500 uppercase tracking-wider">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-slate-200">
            {unifiedData.map((row) => (
              <tr
                key={row.id}
                className={`hover:bg-slate-50 transition cursor-pointer ${
                  selectedRow === row.id ? 'bg-indigo-50 hover:bg-indigo-50' : ''
                }`}
                onClick={() => setSelectedRow(row.id)}
              >
                {/* Client Identity */}
                <td className="px-6 py-4 whitespace-nowrap">
                  <div className="text-sm font-medium text-slate-900">{row.client}</div>
                  <div className="text-xs text-slate-500">{row.contact}</div>
                </td>

                {/* CRM Data */}
                <td className="px-6 py-4 whitespace-nowrap">
                  <div className="text-sm text-slate-700">{row.crm_stage}</div>
                  <div className="w-24">
                    <ProbabilityBar percent={row.probability} />
                  </div>
                </td>

                {/* Marketing Data */}
                <td className="px-6 py-4 whitespace-nowrap">
                  <div className="flex items-center text-sm text-slate-600">
                    <ArrowUpRight className="w-3 h-3 mr-1 text-slate-400" />
                    {row.marketing_touch}
                  </div>
                </td>

                {/* ERP Financial Data */}
                <td className="px-6 py-4 whitespace-nowrap">
                  <div className="flex flex-col items-start">
                    <span className="text-sm font-mono text-slate-900">
                      ${row.erp_balance.toLocaleString('en-US', { minimumFractionDigits: 2 })}
                    </span>
                    <StatusPill status={row.erp_status} days={row.erp_days_overdue} />
                  </div>
                </td>

                {/* Actions */}
                <td className="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                  <button className="text-slate-400 hover:text-indigo-600">
                    <MoreHorizontal className="w-5 h-5" />
                  </button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>

        {/* Footer / Pagination Area */}
        <div className="bg-slate-50 px-6 py-3 border-t border-slate-200 flex justify-between items-center">
          <span className="text-xs text-slate-500">Showing 4 of 12 records</span>
          <div className="flex gap-2">
            <button className="text-xs text-slate-600 border rounded px-2 py-1 bg-white hover:bg-slate-100">Previous</button>
            <button className="text-xs text-slate-600 border rounded px-2 py-1 bg-white hover:bg-slate-100">Next</button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## Component Architecture Breakdown

### Data Flow

```
┌─────────────────┐
│  GraphQL Query  │
│  (Apollo/RQ)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  unifiedData    │ ← In production, fetched from API
│  (State)        │   In this example, hardcoded mock
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  GrowthTable    │
│  Component      │
└────────┬────────┘
         │
         ├──► StatusPill (sub-component)
         ├──► ProbabilityBar (sub-component)
         └──► Row click → setSelectedRow (triggers Context Rail)
```

### State Management

**Local State:**

- `selectedRow`: Tracks which row is currently selected (for highlighting and triggering Context Rail)

**Future State (Production):**

- `filterState`: Active filters (e.g., "Show only overdue")
- `sortState`: Current sort column and direction
- `pageState`: Current page number for pagination

---

## Design Decisions Explained

### Why Mono-Spaced Font for Balance Column?

```jsx
<span className="text-sm font-mono text-slate-900">
  ${row.erp_balance.toLocaleString('en-US', { minimumFractionDigits: 2 })}
</span>
```

**Rationale:** 

In ERP systems, vertical alignment of decimal points is critical for rapid scanning of financial figures. Monospaced fonts (like `font-mono` which maps to Roboto Mono) ensure that:

```
$12,450.00
$   125.50
$55,000.00
```

All decimal points align vertically, making it easier to compare values at a glance.

### Why Visual Probability Bar Instead of Text?

```jsx
<ProbabilityBar percent={row.probability} />
```

**Rationale:**

A visual progress bar conveys win probability **faster than reading a number**. The eye can instantly assess:

- Deal close to 100%? Bar is nearly full (green)
- Deal below 50%? Bar is half-filled (blue)

This reduces cognitive load when scanning multiple rows.

### Why Include "Last Login" Context?

```jsx
last_login: 'Today',
```

**Rationale:**

Combining **product usage data** (Last Login) with **financial data** (Open Balance) helps identify churn risks:

- Customer hasn't logged in + Has overdue invoice = **High churn risk**
- Customer logs in daily + Has overdue invoice = **Likely a billing issue, not churn**

This cross-system insight is only possible when data is unified.

---

## Contextual Logic: Acme Corp Example

Let's analyze the **Acme Corp** row to understand the power of unified data:

```jsx
{
  client: 'Acme Corp',
  crm_stage: 'Negotiation',        // Sales context
  probability: 80,                  // High likelihood of close
  marketing_touch: 'CEO Podcast',   // Recent engagement
  erp_balance: 12450.00,           // Outstanding balance
  erp_status: 'Overdue',           // RED FLAG
  erp_days_overdue: 15,            // 15 days late
  last_login: 'Today',             // Active user
}
```

**What This Tells Us:**

1. **Sales Perspective:** Deal is in Negotiation stage (80% probability) → Salesperson is likely working on closing a new contract
2. **Finance Red Flag:** Customer owes $12,450 and is 15 days overdue → Collections team should be following up
3. **Risk Insight:** Closing a new deal with a customer who hasn't paid existing invoices is risky. Sales should:
    - Coordinate with finance before proceeding
    - Consider requiring prepayment for new order
    - Address payment delay as part of negotiation
4. **Positive Signal:** Customer logged in today → They're actively using the product, so this is likely a billing issue, not churn

**Action:** The salesperson can click the row to open the Context Rail, see the overdue invoice details, and use the "Resend Invoice" quick action button—all without leaving the page.

---

## Production Enhancements

### 1. GraphQL Integration

**Replace mock data with real query:**

```jsx
import { useQuery, gql } from '@apollo/client';

const GET_ACTIVE_PIPELINE = gql`
  query GetActivePipeline {
    activePipeline {
      id
      client
      contact
      crmStage
      probability
      marketingTouch
      erpBalance
      erpStatus
      erpDaysOverdue
      lastLogin
    }
  }
`;

export default function GrowthTable() {
  const { loading, error, data } = useQuery(GET_ACTIVE_PIPELINE, {
    pollInterval: 120000, // Refresh every 2 minutes
  });

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorBanner message={error.message} />;

  return (
    // ... rest of component using data.activePipeline
  );
}
```

### 2. Advanced Filtering

**Add filter controls:**

```jsx
const [filters, setFilters] = useState({
  showOverdueOnly: false,
  minProbability: 0,
  stages: [],
});

const filteredData = unifiedData.filter(row => {
  if (filters.showOverdueOnly && row.erp_status !== 'Overdue') return false;
  if (row.probability < filters.minProbability) return false;
  if (filters.stages.length > 0 && !filters.stages.includes(row.crm_stage)) return false;
  return true;
});
```

### 3. Column Sorting

**Make headers clickable:**

```jsx
const [sortConfig, setSortConfig] = useState({ key: null, direction: 'asc' });

const handleSort = (key) => {
  setSortConfig({
    key,
    direction: sortConfig.key === key && sortConfig.direction === 'asc' ? 'desc' : 'asc',
  });
};

const sortedData = [...filteredData].sort((a, b) => {
  if (!sortConfig.key) return 0;
  const aVal = a[sortConfig.key];
  const bVal = b[sortConfig.key];
  return sortConfig.direction === 'asc' ? aVal - bVal : bVal - aVal;
});
```

### 4. Row Click Integration with Context Rail

**Trigger Context Rail on row click:**

```jsx
import ContextRail from './ContextRail';

export default function GrowthTable() {
  const [selectedRow, setSelectedRow] = useState(null);
  const [railOpen, setRailOpen] = useState(false);

  const handleRowClick = (rowId) => {
    setSelectedRow(rowId);
    setRailOpen(true);
  };

  return (
    <>
      {/* Table component */}
      <ContextRail
        isOpen={railOpen}
        clientId={selectedRow}
        onClose={() => setRailOpen(false)}
      />
    </>
  );
}
```

---

## Accessibility Features

### Keyboard Navigation

```jsx
<tr
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      setSelectedRow(row.id);
    }
  }}
  tabIndex={0}
  role="button"
  aria-label={`View details for ${row.client}`}
>
```

### Screen Reader Support

```jsx
<th scope="col" aria-label="Client name and primary contact">
  Client / Contact
</th>
```

---

## Performance Optimization

### Virtualization for Large Datasets

For tables with 1000+ rows, use `react-virtual` for windowing:

```jsx
import { useVirtual } from 'react-virtual';

const parentRef = useRef();
const rowVirtualizer = useVirtual({
  size: unifiedData.length,
  parentRef,
  estimateSize: useCallback(() => 56, []), // Row height in px
});
```

---

## Testing Strategy

**Unit Tests:**

- StatusPill renders correct color and icon for each status
- ProbabilityBar width matches percentage
- Row selection state updates correctly

**Integration Tests:**

- GraphQL query fetches data correctly
- Filters apply to data without errors
- Sort toggles between ascending and descending

**Visual Regression Tests:**

- Screenshot comparison to ensure UI doesn't break
- Test dark mode rendering

---

## Next Steps

With the Growth Table component complete, proceed to:

- **Subpage 05:** Adaptive Context Rail Component (slide-out detail panel)
- **Subpage 06:** Smart RMA Modal & Transaction Logic (complex form handling)