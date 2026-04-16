---
name: write-ui-spec
description: Create a UI specification through codebase exploration, user interview, and structured page/component design. Use when user wants to plan UI, design pages, create a ui.md, spec out frontend, plan a dashboard/page layout, or produce desktop/mobile wireframes or simple UI flow diagrams for an existing feature.
---

# Write a UI Spec

Create a UI specification document (`ui.md`) for a feature by exploring the existing frontend architecture, interviewing the user about page structure and access control, and producing a structured spec.

The spec should now cover both structure and low-fidelity layout:

- Information architecture and component structure
- Desktop and mobile wireframe diagrams
- Responsive layout notes
- Simple Mermaid diagrams for page/view flows when they help clarify navigation or multi-step interactions

## Process

### 1. Gather context

If a PRD or plan exists for the feature, read it first. Ask the user for a brain dump of what they want the UI to look like — pages, who sees what, key interactions.

### 2. Explore the frontend codebase

Before asking questions, explore to understand:
- Route structure and page organization
- Auth and role-based access patterns
- Navigation structure (sidebar, tabs, breadcrumbs)
- Shared layout and UI components (what's reusable)
- Feature folder conventions
- Similar features that can serve as prior art

This grounds the interview in what actually exists.

### 3. Interview the user

Walk through each page/view one at a time. For each, resolve:
- **Where does it live?** (new route, tab on existing page, section within existing view)
- **Who has access?** (roles, permissions, feature gates)
- **What does it contain?** (tables, forms, charts, cards, read-only info)
- **What actions are available?** (CRUD, bulk operations, exports, copy)
- **How do related entities appear?** (inline, expandable, side panel, dialog, separate page)
- **How does desktop differ from mobile?** (stacking, hidden columns, drawer vs sidebar, sticky actions, tab collapse, etc.)

Provide your recommended answer for each question. Resolve dependencies between decisions before moving on (e.g., settle where the page lives before discussing its contents).

### 4. Write the spec

Save to `.plans/<feature-name>/ui.md`. Use the template below.

<ui-spec-template>

# [Feature Name] — UI Specification

> Brief description of what this spec covers and its scope.

## Feature Gate

- Feature flag name (if applicable) and behavior when off

---

## Page N: [Page Title] (`/route`)

**Access:** [roles/permissions]

**Location:** Where in the app this lives (new sidebar entry, tab on existing page, nested route, etc.)

**Layout:** Layout pattern or component used.

**Responsive Notes:** Key desktop/mobile differences, including layout shifts, collapsed navigation, overflow behavior, and action placement.

### Flow Diagram

```mermaid
flowchart TD
  A["Entry point"] --> B["Page or section"]
  B --> C["Primary action"]
```

Use a simple Mermaid flow only when it adds clarity. Prefer one diagram per page or flow, not per tiny interaction.

### Desktop Wireframe

Render the desktop wireframe as a Mermaid diagram, not as descriptive bullet points. Use simple layout regions and labels. For example:

```mermaid
flowchart TD
  Header["Header: Title | Breadcrumbs | Primary CTA"]
  Body["Desktop Body"]
  Sidebar["Left Sidebar: Filters"]
  Main["Main Content: Summary Cards + Table"]
  Detail["Right Panel: Detail View"]

  Header --> Body
  Body --> Sidebar
  Body --> Main
  Body --> Detail
```

Prefer clarity over visual cleverness. The goal is a low-fidelity structural wireframe that communicates hierarchy and placement.

### Mobile Wireframe

Render the mobile wireframe as a Mermaid diagram, not as descriptive bullet points. Show stacking order and condensed navigation. For example:

```mermaid
flowchart TD
  AppBar["Top App Bar: Title | Overflow"]
  Summary["Summary Cards"]
  Filters["Filters Bottom Sheet Trigger"]
  List["Stacked Entity Cards"]
  Sticky["Sticky Primary Action"]

  AppBar --> Summary
  Summary --> Filters
  Filters --> List
  List --> Sticky
```

Use the mobile wireframe to make layout collapse and action placement obvious at a glance.

### Section/Tab N.N: [Name]

Description of what this section contains.

**Content:** What data is displayed and how (table with columns, card grid, form fields, chart type, summary stats, etc.). Be specific about fields/columns and their formatting.

**Filters/Controls:** Any filtering, sorting, search, date range pickers, or toggle controls.

**Actions:** Available user actions and how they're triggered (buttons, row actions, bulk selection, context menus).

**Create/Edit flows:** How creation and editing work (dialog, sheet/panel, inline, separate page). Specify fields.

**Reuse:** Specific existing components to reuse from the codebase.

---

## Component Summary

### New Components to Create

| Component | Location | Description |
|-----------|----------|-------------|

### Existing Components to Reuse

| Component | Source | Used For |
|-----------|--------|----------|

---

## Access Control Summary

| Page / Component | role1 | role2 | ... |
|-----------------|-------|-------|-----|

---

## Navigation Changes

New entries in sidebar, tab bars, or other navigation surfaces. Include placement order and gating conditions.

</ui-spec-template>

## Key Principles

- **Reuse first.** Check for existing shared components and layout primitives before proposing new ones.
- **Be specific about content and actions.** Vague specs lead to implementation guesswork. Name the fields, columns, and buttons.
- **Resolve sub-entity display patterns explicitly.** "Does this open a dialog, a side panel, or a new page?" is a design decision.
- **Feature-gate new navigation items** when the feature may be rolled out incrementally.
- **Separate management views from consumer views.** Admin CRUD and end-user read-only views are distinct, even if they share components.
- **Wireframe as diagrams, not prose.** Desktop and mobile wireframes should be Mermaid diagrams so the layout can be scanned visually.
- **Wireframe, don't art direct.** Keep the wireframes low-fidelity and structural, not polished visual design directions.
- **Use diagrams selectively.** Add simple Mermaid flows when page transitions or multi-step interactions would otherwise be ambiguous.
