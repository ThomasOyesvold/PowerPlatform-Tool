# Requirements: Power Platform Roadmap Planner

**Defined:** 2026-02-19
**Core Value:** Help Power Platform consultants scope, plan, and deliver implementations consistently — reducing pre-sales time from days to hours, eliminating dependency mistakes, and standardizing client communication quality.

---

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Project Management

- [ ] **PROJ-01**: User can create a new client project with name, description, and client info
- [ ] **PROJ-02**: User can edit project details (name, description, client info)
- [ ] **PROJ-03**: User can delete a project
- [ ] **PROJ-04**: Multiple consultants can edit the same project (near-real-time sync on save)
- [ ] **PROJ-05**: Project changes are versioned — user can view change history
- [ ] **PROJ-06**: User can restore a previous version of a project

### Planning & Sequencing

- [ ] **PLAN-01**: User can define phases/milestones for a project
- [ ] **PLAN-02**: User can mark dependencies between components (e.g., "Power App depends on Dataverse table X")
- [ ] **PLAN-03**: User can visually reorder components using drag-and-drop sequencing
- [ ] **PLAN-04**: System highlights critical path (longest dependency chain) visually
- [ ] **PLAN-05**: User can assign components to phases

### Power Platform Component Design

- [ ] **COMP-01**: User can define Dataverse tables (name, display name, description)
- [ ] **COMP-02**: User can add columns to Dataverse tables (name, type, required, description)
- [ ] **COMP-03**: User can define relationships between Dataverse tables (1:N, N:1, N:N)
- [ ] **COMP-04**: User can define SharePoint Lists (name, description)
- [ ] **COMP-05**: User can add columns to SharePoint Lists (name, type, required, description)
- [ ] **COMP-06**: User can define Power Apps screens (name, description, type: canvas/model-driven)
- [ ] **COMP-07**: User can map navigation flow between Power Apps screens (screen A → screen B on button click)
- [ ] **COMP-08**: User can define Power Automate flows (name, trigger type, description)
- [ ] **COMP-09**: User can map connections between flows and other components (flow triggered by Dataverse create, flow updates SharePoint list)

### Governance & Standards

- [ ] **GOV-01**: User can add connectors to the project (e.g., SharePoint, Office 365 Users, SQL Server)
- [ ] **GOV-02**: System flags when a connector requires Premium licensing
- [ ] **GOV-03**: User can define environment strategy (number of environments, names, purpose: Dev/Test/Prod)
- [ ] **GOV-04**: User can define ALM/promotion flow between environments
- [ ] **GOV-05**: User can generate naming conventions for the project (prefix, format, examples)
- [ ] **GOV-06**: System enforces naming conventions across all components (Power Apps, flows, tables)
- [ ] **GOV-07**: User can export naming convention reference as a document
- [ ] **GOV-08**: System recommends SharePoint Lists vs Dataverse based on project complexity (small/medium projects → SharePoint Lists; large/complex → Dataverse)

### Templates & AI

- [ ] **TMPL-01**: System provides a template library with common Power Platform scenarios (approval flow, form replacement, data migration, etc.)
- [ ] **TMPL-02**: User can browse and preview templates
- [ ] **TMPL-03**: User can apply a template to their project — template components are inserted and editable
- [ ] **TMPL-04**: AI suggests phases based on project components
- [ ] **TMPL-05**: AI suggests component dependencies based on relationships
- [ ] **TMPL-06**: AI suggests sequencing order (what to build first)
- [ ] **TMPL-07**: User can manually override any AI suggestion (delete, edit, ignore)

### Branding & Presentation

- [ ] **BRAND-01**: System provides pre-made color palettes (light/dark themes, corporate styles)
- [ ] **BRAND-02**: User can customize color palette (primary, secondary, accent colors)
- [ ] **BRAND-03**: User can apply color palette to visual outputs (roadmap, diagrams)
- [ ] **PRES-01**: System generates visual stakeholder roadmap (Gantt/timeline view, high-level phases)
- [ ] **PRES-02**: System generates technical delivery plan (detailed component list with dependencies, owners, estimates)
- [ ] **PRES-03**: User can export visual stakeholder roadmap as PDF
- [ ] **PRES-04**: User can export technical delivery plan as PDF

### Collaboration & Export

- [ ] **COLLAB-01**: Changes from one consultant sync to other consultants on save (near-real-time, not live)
- [ ] **COLLAB-02**: User can see who last edited each section of the project
- [ ] **EXPORT-01**: User can export naming convention reference as Word/PDF
- [ ] **EXPORT-02**: User can export Dataverse schema diagram as PNG/PDF
- [ ] **EXPORT-03**: User can export Power Automate flow map as PNG/PDF
- [ ] **EXPORT-04**: User can export Power Apps screen map as PNG/PDF

---

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Advanced Collaboration
- **COLLAB-03**: Real-time collaboration (Google Docs-style live cursors, instant sync)
- **COLLAB-04**: In-app comment system (consultants can discuss components within the tool)
- **COLLAB-05**: @mention notifications

### Advanced Planning
- **PLAN-06**: Effort estimator (t-shirt sizing per component, auto-generate timeline estimates)
- **PLAN-07**: Resource allocation (assign consultants to components, track capacity)
- **PLAN-08**: Progress tracking during delivery (mark components as In Progress / Done)

### Integrations
- **INTEG-01**: Export roadmap to Azure DevOps (create work items from components)
- **INTEG-02**: Export roadmap to Microsoft Planner (create tasks from components)
- **INTEG-03**: Connect to client's SharePoint/Dataverse to read existing structure
- **INTEG-04**: Generate Power Platform solution files directly from roadmap

### Advanced Templates
- **TMPL-08**: User can save their own custom templates for reuse
- **TMPL-09**: Template marketplace (share templates across consultancies)

### Client Portal
- **CLIENT-01**: Client can view roadmap in read-only mode
- **CLIENT-02**: Client can comment on phases/components
- **CLIENT-03**: Client can approve/reject phases

---

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Actual Power Platform development | This is a planning tool, not an IDE — it scopes and designs, doesn't build |
| Automated code generation | Generating Power Apps YAML or Power Automate JSON is brittle and fragile — consultants build manually |
| Billing/invoicing features | Different product — integrate with accounting software instead |
| CRM features (contact management, sales pipeline) | Use Dynamics 365 or existing CRM — not our domain |
| Real-time live collaboration (v1) | Google Docs-style cursors require WebSockets, OT/CRDT — too complex for MVP |
| Template marketplace (v1) | Sharing templates across orgs requires multi-tenancy, moderation, versioning — defer to v2 |

---

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| PROJ-01 | Phase 1 | Pending |
| PROJ-02 | Phase 1 | Pending |
| PROJ-03 | Phase 1 | Pending |
| PROJ-04 | Phase 7 | Pending |
| PROJ-05 | Phase 7 | Pending |
| PROJ-06 | Phase 7 | Pending |
| PLAN-01 | Phase 3 | Pending |
| PLAN-02 | Phase 3 | Pending |
| PLAN-03 | Phase 3 | Pending |
| PLAN-04 | Phase 3 | Pending |
| PLAN-05 | Phase 3 | Pending |
| COMP-01 | Phase 2 | Pending |
| COMP-02 | Phase 2 | Pending |
| COMP-03 | Phase 2 | Pending |
| COMP-04 | Phase 2 | Pending |
| COMP-05 | Phase 2 | Pending |
| COMP-06 | Phase 2 | Pending |
| COMP-07 | Phase 2 | Pending |
| COMP-08 | Phase 2 | Pending |
| COMP-09 | Phase 2 | Pending |
| GOV-01 | Phase 4 | Pending |
| GOV-02 | Phase 4 | Pending |
| GOV-03 | Phase 4 | Pending |
| GOV-04 | Phase 4 | Pending |
| GOV-05 | Phase 4 | Pending |
| GOV-06 | Phase 4 | Pending |
| GOV-07 | Phase 4 | Pending |
| GOV-08 | Phase 4 | Pending |
| TMPL-01 | Phase 5 | Pending |
| TMPL-02 | Phase 5 | Pending |
| TMPL-03 | Phase 5 | Pending |
| TMPL-04 | Phase 5 | Pending |
| TMPL-05 | Phase 5 | Pending |
| TMPL-06 | Phase 5 | Pending |
| TMPL-07 | Phase 5 | Pending |
| BRAND-01 | Phase 6 | Pending |
| BRAND-02 | Phase 6 | Pending |
| BRAND-03 | Phase 6 | Pending |
| PRES-01 | Phase 6 | Pending |
| PRES-02 | Phase 6 | Pending |
| PRES-03 | Phase 6 | Pending |
| PRES-04 | Phase 6 | Pending |
| COLLAB-01 | Phase 7 | Pending |
| COLLAB-02 | Phase 7 | Pending |
| EXPORT-01 | Phase 8 | Pending |
| EXPORT-02 | Phase 8 | Pending |
| EXPORT-03 | Phase 8 | Pending |
| EXPORT-04 | Phase 8 | Pending |

**Coverage:**
- v1 requirements: 45
- Mapped to phases: 45/45 (100%)
- Unmapped: 0

---

*Requirements defined: 2026-02-19*
*Last updated: 2026-02-20 after adding SharePoint Lists vs Dataverse recommendation*
