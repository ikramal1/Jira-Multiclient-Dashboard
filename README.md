# Pilotage Projets — Multi-Client Jira Dashboard (Power BI)

A Power BI dashboard that consolidates Jira project data across multiple clients into a single tool, letting a manager quickly identify which client needs attention and drill into the details.

## Problem

The manager oversaw several client projects in Jira, each accessed separately. There was no fast way to see, across all clients at once, which one had delays, blocked tickets, or unassigned work.

## Solution

A two-page Power BI report:

### 1. Overview page
Answers one question: *which client needs attention right now?*
- An alert card that dynamically highlights the single most critical client (via a DAX `TOPN` calculation, not hardcoded)
- A client table sorted by criticality, with a traffic-light status (🔴🟡🟢)
- Actual vs. expected progress per client
- Overdue tickets broken down by developer

### 2. Detail page (per client)
Selectable via filter or drillthrough from the Overview table.
- Progress / overdue / unassigned gauges for the selected client
- Ticket status breakdown
- Tickets-created-per-month trend, to spot rising workload early
- A table of tickets requiring action, each tagged with why (overdue, unassigned, or both)

## Data model

Star schema with three tables:
- **Tickets** (fact table) — status, priority, creation/due/close dates, links to client and developer
- **Clients** (dimension) — name, start date, delivery date, actual/expected progress
- **Developers** (dimension) — developer names

Relationships: `Tickets[ClientID] → Clients[ClientID]`, `Tickets[DeveloppeurID] → Developpeurs[DeveloppeurID]` (many-to-one).

## Key DAX measures

| Measure | Purpose |
|---|---|
| `JoursRestants` | Days remaining until delivery date |
| `TicketsEnRetard` | Count of non-"Done" tickets past their due date |
| `TicketsNonAssignes` | Count of tickets with no developer assigned |
| `Ecart` | Actual progress minus expected progress — comparable across clients regardless of timeline |
| `StatutClient` | Composite traffic-light status combining progress gap **and** days remaining (catches cases where a client looks fine on progress alone but is close to deadline) |
| `ClientCritique` | Dynamically identifies the single most urgent client via `TOPN` on a composite score |
| `RaisonAction` (calculated column) | Labels each ticket as overdue, unassigned, both, or neither |

## Data source

Currently: manual Excel exports from Jira (`Tickets.xlsx`, `Clients.xlsx`, `Developpeurs.xlsx`).

Planned improvement: direct Power Query connection to the Jira REST API to remove the manual export step and keep data current automatically.

## Design decisions

- Two pages instead of three, to keep the report maintainable
- Overview leads with a single dynamic "client to watch" card rather than only aggregate KPIs — the real question was "who," not "how are we doing on average"
- No filters on the Overview page by design, to preserve its at-a-glance purpose; Client/Priority/Developer filters live on the Detail page instead
- A "workload by developer" ticket-count chart was deliberately replaced with an action-oriented table, since raw ticket counts don't reflect actual workload (priority matters more than volume)
- Consistent branding (navigation bar, color palette, card styling) applied throughout for a polished, app-like feel rather than a default Power BI look

## Files

- `dashboard.pbix` — the Power BI report
- `data/` — source Excel files
- `README.md` — this file

## Status / next steps

- [ ] Connect directly to Jira API instead of manual Excel export
- [ ] Replace the stacked bar chart with a Sankey visual (Client → Status → Priority) once AppSource access is available
- [ ] Revisit criticality thresholds with the manager based on real-world judgment of what counts as urgent
