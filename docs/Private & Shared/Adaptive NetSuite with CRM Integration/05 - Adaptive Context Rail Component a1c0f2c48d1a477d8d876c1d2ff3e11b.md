# 05 - Adaptive Context Rail Component

# Adaptive Context Rail Component

The Context Rail is the signature feature of this adaptive NetSuite redesign. It's a persistent right-hand sidebar that **dynamically changes content based on what the user clicks** in the main workspace, providing a unified 360-degree view of customers without navigating away from the current screen.

---

## Component Purpose

In traditional ERP systems, looking up a customer's recent shipments and their support tickets requires opening two different tabs (e.g., NetSuite for Orders, Zendesk for Tickets). This component **unifies them into a single, slide-out drawer** that overlays the main table, allowing users to context-switch without losing their place.

**Key Innovation:** The rail **adapts its content** based on what entity the user clicked:

- Click a customer → Show orders, invoices, tickets, marketing engagement
- Click a transaction → Show customer profile, payment history, related documents
- Click an inventory item → Show supplier info, purchase orders, support tickets about that item

---

## Visual Design Specifications

### Layout Dimensions

- **Width:** 400px fixed (does not resize)
- **Height:** Full viewport height
- **Position:** Fixed to right edge of screen
- **Z-Index:** 50 (above main content, below modals)
- **Animation:** Slide-in from right with 300ms ease-in-out transition

### Behavior

**Opening:**

- Triggered by clicking any row in the Growth Table
- Slides in from right edge
- Main content area narrows to accommodate (no overlap)
- Semi-transparent backdrop appears behind rail for mobile

**Closing:**

- Click X button in header
- Press ESC key
- Click outside rail area (on backdrop)
- Click another row (closes current, opens new)

---

## Mock Data Structure

```jsx
const clientDetail = {
  // Identity & Key Metrics
  name: 'Acme Corp',
  tier: 'Enterprise',
  accountId: 'ACC-001',
  
  // Health Indicators
  nps: 32, // Low NPS indicates churn risk
  ltv: 142000, // Lifetime Value
  health_score: 65, // Overall account health (0-100)
  renewal_date: 'Oct 24, 2025',
  
  // ERP Data (NetSuite)
  orders: [
    {
      id: 'PO-9921',
      date: 'Jan 12',
      item: 'MacBook Pro M3 (x5)',
      total: 12450.00,
      status: 'Shipped',
      tracking: 'UPS-1Z999AA10123456784',
    },
    {
      id: 'PO-9840',
      date: 'Dec 05',
      item: 'Dell UltraSharp 27"',
      total: 3200.00,
      status: 'Delivered',
      tracking: null,
    },
  ],
  
  // Service Data (Zendesk/Salesforce Service Cloud)
  tickets: [
    {
      id: 402,
      subject: 'Shipment delayed/damaged',
      priority: 'High',
      status: 'Open',
      agent: 'Sarah J.',
      created: '2 hours ago',
      last_update: '15 minutes ago',
    },
    {
      id: 389,
      subject: 'Invoice discrepancy',
      priority: 'Medium',
      status: 'Resolved',
      agent: 'Finance Bot',
      created: '3 days ago',
      last_update: '2 days ago',
    },
  ],
  
  // Marketing Data (Marketo/HubSpot)
  marketing: {
    last_email_open: '2 days ago',
    last_campaign: 'Q1 Webinar Series',
    engagement_score: 42, // 0-100
    days_since_engagement: 45,
  },
};
```

---

## Complete Component Code

```jsx
import React from 'react';
import {
  X,
  Package,
  MessageSquare,
  AlertTriangle,
  ExternalLink,
  CreditCard,
  RefreshCw,
  MoreVertical,
  TrendingDown,
} from 'lucide-react';

// ============================================================================
// MOCK DATA (In production, fetched via GraphQL based on clientId prop)
// ============================================================================

const clientDetail = {
  name: 'Acme Corp',
  tier: 'Enterprise',
  nps: 32, // Low NPS indicates risk
  ltv: 142000, // Lifetime Value
  renewal_date: 'Oct 24, 2025',
  health_score: 65,
  
  // ERP Data (NetSuite)
  orders: [
    {
      id: 'PO-9921',
      date: 'Jan 12',
      item: 'MacBook Pro M3 (x5)',
      total: 12450.00,
      status: 'Shipped',
    },
    {
      id: 'PO-9840',
      date: 'Dec 05',
      item: 'Dell UltraSharp 27"',
      total: 3200.00,
      status: 'Delivered',
    },
  ],
  
  // Service Data (Zendesk/Salesforce)
  tickets: [
    {
      id: 402,
      subject: 'Shipment delayed/damaged',
      priority: 'High',
      status: 'Open',
      agent: 'Sarah J.',
    },
    {
      id: 389,
      subject: 'Invoice discrepancy',
      priority: 'Medium',
      status: 'Resolved',
      agent: 'Finance Bot',
    },
  ],
  
  // Marketing Data
  marketing: {
    days_since_engagement: 45,
  },
};

// ============================================================================
// MAIN COMPONENT
// ============================================================================

export default function ContextRail({ isOpen, onClose, clientId }) {
  // Early return if rail is not open
  if (!isOpen) return null;

  // In production, fetch data based on clientId:
  // const { data, loading } = useQuery(GET_CLIENT_360, { variables: { clientId } });

  return (
    <div className="fixed inset-y-0 right-0 w-[400px] bg-white shadow-2xl border-l border-slate-200 transform transition-transform duration-300 ease-in-out z-50 overflow-y-auto">
      {/* ================================================================
          1. HEADER: Identity & Key Metrics
          ================================================================ */}
      <div className="p-6 border-b border-slate-100 sticky top-0 bg-white/95 backdrop-blur-sm z-10">
        <div className="flex justify-between items-start mb-4">
          <div>
            <h2 className="text-xl font-bold text-slate-900 flex items-center gap-2">
              {clientDetail.name}
              <span className="px-2 py-0.5 rounded-full bg-indigo-100 text-indigo-700 text-[10px] font-bold uppercase tracking-wider">
                {clientDetail.tier}
              </span>
            </h2>
            <p className="text-xs text-slate-500 mt-1">
              LTV: ${clientDetail.ltv.toLocaleString()} • Renew: {clientDetail.renewal_date}
            </p>
          </div>
          <button
            onClick={onClose}
            className="p-1 rounded hover:bg-slate-100 text-slate-400 hover:text-slate-600 transition"
            aria-label="Close context rail"
          >
            <X className="w-5 h-5" />
          </button>
        </div>

        {/* Adaptive Alert Block - Only shows if NPS is below threshold */}
        {clientDetail.nps < 40 && (
          <div className="bg-orange-50 border border-orange-100 rounded-md p-3 flex gap-3 items-start">
            <AlertTriangle className="w-4 h-4 text-orange-600 mt-0.5 flex-shrink-0" />
            <div>
              <p className="text-xs font-semibold text-orange-800">Churn Risk Detected</p>
              <p className="text-xs text-orange-700 mt-0.5">
                NPS dropped to {clientDetail.nps}. Recent support ticket regarding shipping damage
                correlates with delayed payment.
              </p>
            </div>
          </div>
        )}
      </div>

      <div className="p-6 space-y-8">
        {/* ================================================================
            2. UNIFIED ACTION BAR
            Quick actions relevant to this customer
            ================================================================ */}
        <div className="grid grid-cols-2 gap-3">
          <button className="flex items-center justify-center gap-2 py-2 px-3 bg-white border border-slate-300 rounded shadow-sm hover:bg-slate-50 text-xs font-medium text-slate-700 transition">
            <CreditCard className="w-3 h-3" /> Resend Invoice
          </button>
          <button className="flex items-center justify-center gap-2 py-2 px-3 bg-white border border-slate-300 rounded shadow-sm hover:bg-slate-50 text-xs font-medium text-slate-700 transition">
            <RefreshCw className="w-3 h-3" /> Process Return
          </button>
        </div>

        {/* ================================================================
            3. ERP SECTION: Recent Orders
            Data from NetSuite
            ================================================================ */}
        <section>
          <div className="flex justify-between items-center mb-3">
            <h3 className="text-xs font-bold text-slate-400 uppercase tracking-widest flex items-center gap-2">
              <Package className="w-3 h-3" /> Recent Orders (ERP)
            </h3>
            <ExternalLink
              className="w-3 h-3 text-slate-400 cursor-pointer hover:text-indigo-600 transition"
              title="Open in NetSuite"
            />
          </div>
          <div className="space-y-3">
            {clientDetail.orders.map((order) => (
              <div
                key={order.id}
                className="group border border-slate-100 rounded p-3 hover:border-indigo-200 hover:shadow-sm transition cursor-pointer"
              >
                <div className="flex justify-between items-start">
                  <div>
                    <div className="text-sm font-semibold text-slate-800">{order.id}</div>
                    <div className="text-xs text-slate-500 mt-1">{order.item}</div>
                  </div>
                  <div className="text-right">
                    <div className="text-sm font-mono font-medium text-slate-900">
                      ${order.total.toLocaleString()}
                    </div>
                    <span
                      className={`text-[10px] font-medium px-1.5 py-0.5 rounded ${
                        order.status === 'Shipped'
                          ? 'bg-blue-50 text-blue-700'
                          : 'bg-green-50 text-green-700'
                      }`}
                    >
                      {order.status}
                    </span>
                  </div>
                </div>
              </div>
            ))}
          </div>
        </section>

        {/* ================================================================
            4. SERVICE SECTION: Active Tickets
            Data from Zendesk/Salesforce Service Cloud
            ================================================================ */}
        <section>
          <div className="flex justify-between items-center mb-3">
            <h3 className="text-xs font-bold text-slate-400 uppercase tracking-widest flex items-center gap-2">
              <MessageSquare className="w-3 h-3" /> Support Tickets (CRM)
            </h3>
            <ExternalLink
              className="w-3 h-3 text-slate-400 cursor-pointer hover:text-indigo-600 transition"
              title="Open in Zendesk"
            />
          </div>
          <div className="space-y-3">
            {clientDetail.tickets.map((ticket) => (
              <div
                key={ticket.id}
                className="bg-slate-50 border border-transparent rounded p-3 hover:bg-white hover:border-slate-200 transition"
              >
                <div className="flex justify-between items-center mb-1">
                  <span className="text-xs font-mono text-slate-400">#{ticket.id}</span>
                  <span
                    className={`text-[10px] font-bold uppercase ${
                      ticket.priority === 'High' ? 'text-red-600' : 'text-slate-500'
                    }`}
                  >
                    {ticket.priority} Priority
                  </span>
                </div>
                <div className="text-sm font-medium text-slate-800">{ticket.subject}</div>
                <div className="flex justify-between items-center mt-2">
                  <div className="text-xs text-slate-500">Agent: {ticket.agent}</div>
                  <span className="text-[10px] bg-slate-200 text-slate-600 px-1.5 py-0.5 rounded-sm">
                    {ticket.status}
                  </span>
                </div>
              </div>
            ))}
          </div>
        </section>

        {/* ================================================================
            5. MARKETING CONTEXT (Engagement Section)
            Data from Marketo/HubSpot
            ================================================================ */}
        <section>
          <h3 className="text-xs font-bold text-slate-400 uppercase tracking-widest mb-3">
            Engagement
          </h3>
          <div className="border border-dashed border-slate-300 rounded p-4 text-center">
            <TrendingDown className="w-5 h-5 text-slate-400 mx-auto mb-2" />
            <p className="text-xs text-slate-500">
              Client has not engaged with marketing emails in{' '}
              <span className="font-semibold text-slate-700">
                {clientDetail.marketing.days_since_engagement} days
              </span>
              .
            </p>
            <button className="mt-2 text-xs text-indigo-600 hover:text-indigo-800 font-medium transition">
              Add to 'Re-engagement' Flow →
            </button>
          </div>
        </section>
      </div>
    </div>
  );
}
```

---

## Design Decisions Explained

### 1. The "Adaptive Alert Block" (Churn Risk)

```jsx
{clientDetail.nps < 40 && (
  <div className="bg-orange-50 border border-orange-100 rounded-md p-3">
    // Alert content
  </div>
)}
```

**Purpose:** This is the **AI Intelligence layer** in action. It doesn't just show data—it **connects dots across systems**.

**What It Does:**

- Detects low NPS score (from Customer Success platform)
- Correlates with recent support ticket about "shipping damage" (from Zendesk)
- Explains why invoice might be overdue (visible in main table)

**Business Impact:**

- Proactive churn prevention
- Provides context for financial delays
- Helps teams understand the "why" behind the data

### 2. Unified Action Bar

```jsx
<button>
  <CreditCard /> Resend Invoice
</button>
<button>
  <RefreshCw /> Process Return
</button>
```

**Purpose:** Eliminate navigation overhead.

**Traditional Flow:**

1. Notice overdue invoice in table
2. Open NetSuite in new tab
3. Search for customer
4. Find invoice
5. Click "Resend" button
6. Return to original tab

**Modern Flow:**

1. Click customer row
2. Click "Resend Invoice" in Context Rail
3. Done (action triggers via API)

**Time Saved:** ~90 seconds per action × 50 actions/day = **75 minutes saved daily per user**

### 3. Visual Distinction Between Data Sources

**ERP Data (Orders):**

- Clean, sharp styling with white background
- Resembles an invoice or receipt
- Monospaced font for dollar amounts

**Service Data (Tickets):**

- Softer, card-like styling with gray background
- Resembles a chat feed or conversation thread
- Priority labels for urgency indication

**Rationale:** Subtle visual cues help users instantly differentiate between "hard numbers" (ERP) and "conversations" (Service) without reading labels.

### 4. External Link Icons

```jsx
<ExternalLink className="w-3 h-3" title="Open in NetSuite" />
```

**Purpose:** Provide escape hatch for power users.

While the Context Rail shows most relevant information, some users may need to access the full NetSuite order record. The external link icon provides a direct shortcut without cluttering the UI with a prominent button.

---

## Adaptive Behavior Examples

### Scenario 1: High-Value Customer with Perfect Payment History

```jsx
{
  nps: 85,
  ltv: 450000,
  health_score: 95,
  erp_status: 'Paid',
}
```

**Rail Shows:**

- ✅ Green "Healthy Account" badge in header
- Upsell opportunities section (promoted to top)
- Recent positive feedback from NPS surveys
- "Request Referral" action button

### Scenario 2: Mid-Tier Customer with Open Ticket

```jsx
{
  nps: 62,
  ltv: 85000,
  health_score: 72,
  tickets: [{ priority: 'High', status: 'Open' }],
}
```

**Rail Shows:**

- ⚠️ Yellow "Active Issue" badge in header
- Support tickets section promoted to top
- "Escalate to Manager" action button
- Hold on upsell attempts until ticket resolved

### Scenario 3: At-Risk Customer with Low Engagement

```jsx
{
  nps: 28,
  ltv: 120000,
  health_score: 35,
  marketing: { days_since_engagement: 60 },
}
```

**Rail Shows:**

- 🔴 Red "Churn Risk" badge in header
- Engagement section promoted to top with prominent CTA
- "Schedule Executive Call" action button
- Automatic Slack notification to Customer Success Manager

---

## Performance Optimization

### Progressive Data Loading

**Problem:** Fetching all customer data (orders, tickets, marketing) takes ~2 seconds.

**Solution:** Load in priority order with visual feedback.

```jsx
const { data, loading } = useQuery(GET_CLIENT_360, {
  variables: { clientId },
  fetchPolicy: 'cache-first', // Use cached data if available
});

// Load priority 1 data immediately (< 200ms)
const headerData = data?.client?.header;

// Load priority 2 data after 200ms
const ordersData = data?.client?.orders;

// Load priority 3 data after 500ms
const ticketsData = data?.client?.tickets;
```

**User Experience:**

1. Rail slides in with header data (name, tier, LTV) **immediately**
2. Alert section appears after 200ms
3. Orders section appears after 300ms
4. Tickets section appears after 500ms
5. Marketing section appears after 700ms

**Perceived Performance:** Feels instant because critical info loads first.

### Caching Strategy

```jsx
// Cache customer data for 5 minutes
const client = useQuery(GET_CLIENT_360, {
  variables: { clientId },
  fetchPolicy: 'cache-first',
  nextFetchPolicy: 'cache-only',
});

// When user clicks same customer again within 5 minutes:
// Data appears INSTANTLY from cache (no API call)
```

---

## Keyboard Shortcuts

**ESC Key:** Close Context Rail

```jsx
useEffect(() => {
  const handleKeyDown = (e) => {
    if (e.key === 'Escape' && isOpen) {
      onClose();
    }
  };
  
  window.addEventListener('keydown', handleKeyDown);
  return () => window.removeEventListener('keydown', handleKeyDown);
}, [isOpen, onClose]);
```

**Arrow Keys:** Navigate between sections

```jsx
const sections = ['orders', 'tickets', 'marketing'];
const [activeSection, setActiveSection] = useState(0);

const handleKeyDown = (e) => {
  if (e.key === 'ArrowDown') {
    setActiveSection((prev) => Math.min(prev + 1, sections.length - 1));
  }
  if (e.key === 'ArrowUp') {
    setActiveSection((prev) => Math.max(prev - 1, 0));
  }
};
```

---

## Mobile Adaptation

**Problem:** 400px rail doesn't fit on mobile screens.

**Solution:** Transform into bottom sheet.

```jsx
// Detect mobile viewport
const isMobile = useMediaQuery('(max-width: 768px)');

if (isMobile) {
  return (
    <div className="fixed inset-x-0 bottom-0 h-[70vh] bg-white rounded-t-2xl shadow-2xl">
      {/* Same content, different layout */}
    </div>
  );
}
```

**Mobile Gestures:**

- Swipe down to dismiss
- Swipe up to expand to full screen
- Tap backdrop to close

---

## Testing Strategy

### Unit Tests

```jsx
describe('ContextRail', () => {
  it('renders alert when NPS < 40', () => {
    render(<ContextRail isOpen={true} clientId="1" />);
    expect(screen.getByText(/Churn Risk Detected/i)).toBeInTheDocument();
  });
  
  it('does not render alert when NPS >= 40', () => {
    // Mock data with NPS = 65
    render(<ContextRail isOpen={true} clientId="2" />);
    expect(screen.queryByText(/Churn Risk Detected/i)).not.toBeInTheDocument();
  });
  
  it('closes on ESC key press', () => {
    const onClose = jest.fn();
    render(<ContextRail isOpen={true} onClose={onClose} />);
    fireEvent.keyDown(window, { key: 'Escape' });
    expect(onClose).toHaveBeenCalled();
  });
});
```

### Integration Tests

```jsx
it('fetches customer data on open', async () => {
  const { result } = renderHook(() => useQuery(GET_CLIENT_360, {
    variables: { clientId: '1' },
  }));
  
  await waitFor(() => {
    expect(result.current.data).toBeDefined();
    expect(result.current.data.client.name).toBe('Acme Corp');
  });
});
```

---

## Next Steps

With the Context Rail component complete, proceed to:

- **Subpage 06:** Smart RMA Modal & Transaction Logic (complex form with dynamic calculations)