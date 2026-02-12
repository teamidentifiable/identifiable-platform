# 01 - Design Philosophy & NetSuite Canvas

# Design Philosophy: "NetSuite Canvas"

This section defines the foundational design principles that guide the entire adaptive NetSuite redesign. The philosophy moves away from static, tab-heavy interfaces toward a modular, canvas-based approach inspired by modern tools like Retool.

---

## Core Concept: Block-Based UI Architecture

Instead of static pages, the UI is built on **"Blocks"**—interactive, resizable containers similar to Retool modules. Each block is a self-contained unit that can be:

- **Dragged and repositioned** across the canvas
- **Resized** to prioritize different data views
- **Customized** with filters, sorts, and display preferences
- **Saved** as user-specific dashboard layouts

<aside>
💡

**Design Goal:** Enable every user to create their own "mission control" by arranging blocks (Profit/Loss charts, Pipeline views, Inventory widgets) according to their specific role and workflow needs.

</aside>

---

## Visual Style Specifications

### Background & Container System

**Light Mode (Standard)**

- **Background Color:** `#F9FAFB` (Off-white, reduces eye strain)
- **Container Background:** `#FFFFFF` (Pure white cards)
- **Drop Shadows:** Subtle `0 1px 3px rgba(0, 0, 0, 0.1)` for depth
- **Borders:** `1px solid #E5E7EB` (Light gray, barely visible)
- **Padding Strategy:** Compact (12-16px) to maximize screen real estate

**Dark Mode**

- **Background Color:** `#111827` (Deep slate, true black causes burn-in)
- **Container Background:** `#1F2937` (Dark gray cards)
- **Drop Shadows:** Subtle `0 1px 3px rgba(0, 0, 0, 0.3)` for contrast
- **Borders:** `1px solid #374151` (Medium gray for visibility)
- **Text Adjustments:** Slightly reduced contrast to prevent glare

### Spacing & Density Philosophy

**High Information Density**

- Compact padding to show more data per screen
- Minimal vertical spacing between rows in tables
- Strategic use of whitespace only where it enhances comprehension
- Dense grids (12-column system) for flexible layouts

**Whitespace Strategy**

- Use whitespace to create visual groupings, not decoration
- Increase padding around interactive elements (buttons, inputs) for touch targets
- Maintain 8px baseline grid for consistent rhythm

---

## Typography System

### Font Selection Rationale

**Inter (Sans-Serif) - Primary Interface Font**

- **Use Cases:** Headers, labels, body text, navigation
- **Rationale:** Excellent readability at small sizes, extensive weight options
- **Weights Used:**
    - 400 (Regular) - Body text
    - 500 (Medium) - UI labels, buttons
    - 600 (Semi-Bold) - Subheadings
    - 700 (Bold) - Main headings, emphasis

**Roboto Mono (Monospaced) - Data & Numbers**

- **Use Cases:** Financial figures, IDs, timestamps, code
- **Rationale:** Vertical alignment of decimal points, clear digit differentiation
- **Weights Used:**
    - 400 (Regular) - Standard data display
    - 500 (Medium) - Highlighted values

### Type Scale

| Element | Font | Size | Weight | Line Height |
| --- | --- | --- | --- | --- |
| Page Title | Inter | 24px | 700 | 32px |
| Section Header | Inter | 18px | 600 | 24px |
| Card Title | Inter | 14px | 600 | 20px |
| Body Text | Inter | 14px | 400 | 20px |
| UI Label | Inter | 12px | 500 | 16px |
| Data/Numbers | Roboto Mono | 14px | 400 | 20px |
| Small Text | Inter | 12px | 400 | 16px |
| Micro Text | Inter | 10px | 500 | 14px |

---

## Interaction Patterns

### Edit Mode Toggle

**Two-State System:**

**1. View Mode (Default)**

- All blocks are locked in position
- Users interact with data (clicking, filtering, sorting)
- Standard cursor behavior
- Optimal for day-to-day operations

**2. Edit Mode**

- Toggle activated via button in top-right corner
- Blocks gain visible handles and borders
- Drag-and-drop repositioning enabled
- Resize handles appear on block corners
- "Save Layout" and "Reset to Default" options appear

**Implementation Notes:**

- Edit mode state persists per user, not per session
- Layouts are saved to user profile via API
- Team admins can create "template layouts" for roles

### Drag-and-Drop Mechanics

**Grid Snapping:**

- 12-column grid system (similar to Bootstrap)
- Blocks snap to grid boundaries when dragged
- Visual grid overlay appears during drag operations
- Minimum block width: 3 columns (25%)
- Maximum block width: 12 columns (100%)

**Drop Zones:**

- Valid drop zones highlight in blue when hovering
- Invalid zones show red border with tooltip explanation
- Other blocks shift dynamically to accommodate dropped block

---

## The "Context Rail" - Adaptive Feature

The Context Rail is the signature feature that makes this design "adaptive." It's a persistent right-hand sidebar that **dynamically changes content based on what the user clicks** in the main workspace.

### Behavior Specifications

**Trigger Mechanism:**

- Clicking any entity (customer, transaction, inventory item) in the main view
- Rail slides in from right with 300ms ease-in-out animation
- Main content area narrows to accommodate (no overlap)
- Close via X button, ESC key, or clicking outside rail

**Contextual Content Loading:**

| User Clicks | Rail Displays |
| --- | --- |
| **Transaction Row** | Customer CRM profile + Marketing touchpoint history + Payment status |
| **Inventory Item** | Supplier health metrics + Recent purchase orders + Open support tickets related to item |
| **Customer Name** | Full 360 view: Orders, Invoices, Tickets, Marketing engagement, NPS score |
| **Campaign Name** | Attributed revenue, spend breakdown, lead sources, conversion funnel |
| **Support Ticket** | Related customer account, order history, previous tickets, escalation path |

### Rail Layout Structure

**Width:** 400px fixed (doesn't resize)

**Sections (Top to Bottom):**

1. **Header** (80px) - Entity name, key identifier, close button
2. **Alert Zone** (variable) - AI-driven warnings, risk indicators
3. **Action Bar** (60px) - Quick action buttons relevant to entity
4. **Data Sections** (scrollable) - Organized by data source (ERP, CRM, Marketing, Service)
5. **Footer** (optional) - Last updated timestamp, data source indicators

---

## Color System for Semantic Meaning

### Status Indicators

**Success States (Green Family)**

- `bg-green-50` / `text-green-800` - Paid invoices, completed orders
- `bg-green-100` / `text-green-700` - Positive trends, healthy metrics

**Warning States (Orange/Yellow Family)**

- `bg-orange-50` / `text-orange-800` - Overdue payments (1-15 days)
- `bg-yellow-50` / `text-yellow-800` - Low inventory, upcoming deadlines

**Error States (Red Family)**

- `bg-red-50` / `text-red-800` - Overdue payments (15+ days), failed transactions
- `bg-red-100` / `text-red-700` - Critical alerts, system errors

**Neutral States (Gray Family)**

- `bg-gray-50` / `text-gray-700` - Pending states, no data
- `bg-gray-100` / `text-gray-600` - Inactive or archived items

**Primary Action (Indigo Family)**

- `bg-indigo-600` / `text-white` - Primary buttons, key CTAs
- `bg-indigo-50` / `text-indigo-700` - Selected states, active filters

### Data Visualization Colors

**Chart Color Palette (Ordered by Priority):**

1. `#6366F1` (Indigo) - Primary metric
2. `#10B981` (Green) - Revenue, positive metrics
3. `#EF4444` (Red) - Expenses, negative metrics
4. `#F59E0B` (Orange) - Secondary metric
5. `#3B82F6` (Blue) - Tertiary metric
6. `#8B5CF6` (Purple) - Additional data series

---

## Responsive Behavior

### Breakpoint Strategy

| Breakpoint | Width | Layout Adjustments |
| --- | --- | --- |
| **Desktop (XL)** | ≥ 1536px | Full 12-column grid, Context Rail visible |
| **Desktop (L)** | 1280-1535px | 12-column grid, Context Rail visible |
| **Desktop (M)** | 1024-1279px | 12-column grid, Context Rail overlays |
| **Tablet** | 768-1023px | 8-column grid, Context Rail overlays |
| **Mobile** | < 768px | Single column, Context Rail becomes bottom sheet |

### Mobile Adaptations

**Dashboard Blocks:**

- Stack vertically in single column
- Full-width cards with collapsible sections
- Swipe gestures for navigation between blocks

**Context Rail:**

- Transforms into bottom sheet (slides up from bottom)
- Covers 70% of screen height
- Swipe down to dismiss

**Tables:**

- Horizontal scroll enabled
- Sticky first column (typically entity name)
- "Expand row" action reveals all columns in card format

---

## Accessibility Standards

### Compliance Targets

- **WCAG 2.1 Level AA** compliance minimum
- **Section 508** compliance for government use cases

### Specific Requirements

**Color Contrast:**

- Text: Minimum 4.5:1 ratio for normal text
- Large text (18px+): Minimum 3:1 ratio
- Interactive elements: Minimum 3:1 ratio against background

**Keyboard Navigation:**

- All interactive elements accessible via Tab key
- Logical tab order following visual flow
- Focus indicators visible (2px indigo outline)
- ESC key closes modals and Context Rail

**Screen Reader Support:**

- Semantic HTML5 elements (`<nav>`, `<main>`, `<aside>`, `<section>`)
- ARIA labels for icon-only buttons
- ARIA live regions for dynamic content updates
- Role attributes for custom components

**Motion & Animation:**

- `prefers-reduced-motion` media query support
- All animations can be disabled via user preference
- No auto-playing videos or carousels

---

## Performance Design Principles

### Lazy Loading Strategy

**Initial Page Load:**

- Load only visible viewport content
- Defer off-screen blocks until scroll
- Lazy load images and charts

**Context Rail:**

- Load header and alert data immediately (< 200ms)
- Progressive load of data sections (prioritize most used)
- Cache previously viewed entities for instant re-display

### Optimistic UI Updates

**Philosophy:** Show the user what they expect to see immediately, then reconcile with server response.

**Example: Creating a Purchase Order**

1. User clicks "Create PO" button
2. UI immediately shows new PO in list with "Pending" badge
3. API call sent to NetSuite in background
4. On success: Badge changes to "Confirmed" with PO number
5. On failure: Row highlights red, inline error message, "Retry" button appears

---

## Design Tokens (CSS Variables)

For consistent theming and easy dark mode implementation:

```css
:root {
  /* Spacing */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
  
  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  
  /* Shadows */
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.15);
  
  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 300ms ease;
  --transition-slow: 500ms ease;
}
```

---

## Next Steps

With this design philosophy established, the following subpages detail the specific implementations:

- **Subpage 02:** Unified Growth Engine Dashboard (data structure and layout)
- **Subpage 03:** Data Architecture & Integration Layer (GraphQL schema)
- **Subpage 04-06:** Component implementations with production-ready code