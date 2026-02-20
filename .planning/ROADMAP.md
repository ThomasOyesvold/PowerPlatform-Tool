# Roadmap: Power Platform Roadmap Planner

**Created:** 2026-02-20
**Depth:** Comprehensive
**Total Phases:** 8

---

## Overview

This roadmap delivers a planning tool for Power Platform consultants in 8 phases. Starting with core project management, building through component design and planning capabilities, then layering governance, AI assistance, presentation, collaboration, and export features. Each phase delivers a complete, verifiable capability.

---

## Phase 1: Foundation - Project Management

**Goal:** Users can create and manage client projects

**Dependencies:** None (foundation)

**Requirements:** PROJ-01, PROJ-02, PROJ-03, AUDIT-01, AUDIT-02, AUDIT-03, AUDIT-04, AUDIT-05, AUDIT-06

**Plans:** 3 plans

Plans:
- [ ] 01-01-PLAN.md — Project scaffolding, Supabase setup, database schema
- [ ] 01-02-PLAN.md — User authentication with login/signup
- [ ] 01-03-PLAN.md — Project CRUD with dashboard and forms

**Success Criteria:**
1. User can create a new project with name, description, and client info
2. User can edit existing project details
3. User can delete a project they created
4. Projects persist across sessions
5. All code includes JSDoc comments and inline explanations
6. Database operations are logged for audit trails
7. Git commits follow conventional commit format with detailed reasoning

---

## Phase 2: Component Design Core

**Goal:** Users can define Power Platform components (tables, lists, apps, flows)

**Dependencies:** Phase 1 (requires projects to exist)

**Requirements:** COMP-01, COMP-02, COMP-03, COMP-04, COMP-05, COMP-06, COMP-07, COMP-08, COMP-09

**Success Criteria:**
1. User can define Dataverse tables with columns, types, and relationships (1:N, N:1, N:N)
2. User can define SharePoint Lists with columns and types
3. User can define Power Apps screens and map navigation flows between them
4. User can define Power Automate flows with triggers and connections to other components
5. All components display in project view with full details

---

## Phase 3: Planning & Sequencing

**Goal:** Users can organize components into phases and define dependencies

**Dependencies:** Phase 2 (requires components to exist)

**Requirements:** PLAN-01, PLAN-02, PLAN-03, PLAN-04, PLAN-05

**Success Criteria:**
1. User can create phases/milestones and assign components to them
2. User can mark dependencies between components (e.g., Power App depends on Dataverse table)
3. User can reorder components using drag-and-drop sequencing
4. System visually highlights the critical path (longest dependency chain)

---

## Phase 4: Governance & Standards

**Goal:** Users can track licensing, environments, and enforce naming conventions

**Dependencies:** Phase 2 (requires components to govern)

**Requirements:** GOV-01, GOV-02, GOV-03, GOV-04, GOV-05, GOV-06, GOV-07, GOV-08

**Success Criteria:**
1. User can add connectors to project and system flags Premium licensing requirements
2. User can define environment strategy (Dev/Test/Prod) with ALM promotion flow
3. User can generate and apply naming conventions across all components
4. System enforces naming conventions when components are created or edited
5. User can export naming convention reference as a document
6. System recommends SharePoint Lists vs Dataverse based on project complexity (budget, scale, data volume)

---

## Phase 5: Templates & AI Assistance

**Goal:** Users can leverage templates and AI to accelerate planning

**Dependencies:** Phase 2 (requires component structure to populate)

**Requirements:** TMPL-01, TMPL-02, TMPL-03, TMPL-04, TMPL-05, TMPL-06, TMPL-07

**Success Criteria:**
1. User can browse template library with common scenarios (approval flow, form replacement, data migration)
2. User can preview template contents before applying
3. User can apply template to project - components are inserted and fully editable
4. AI suggests phases, dependencies, and sequencing based on project components
5. User can manually override, edit, or ignore any AI suggestion

---

## Phase 6: Branding & Visual Presentation

**Goal:** Users can generate professional client-facing roadmaps

**Dependencies:** Phase 2, Phase 3 (requires components and phases to visualize)

**Requirements:** BRAND-01, BRAND-02, BRAND-03, PRES-01, PRES-02, PRES-03, PRES-04

**Success Criteria:**
1. User can select pre-made color palettes or customize their own (primary, secondary, accent)
2. System generates visual stakeholder roadmap (Gantt/timeline view with high-level phases)
3. System generates technical delivery plan (detailed component list with dependencies and owners)
4. User can export both stakeholder roadmap and technical plan as PDFs
5. Color palette applies consistently across all visual outputs

---

## Phase 7: Collaboration

**Goal:** Multiple consultants can work on the same project

**Dependencies:** Phase 1 (extends project management)

**Requirements:** PROJ-04, PROJ-05, PROJ-06, COLLAB-01, COLLAB-02

**Success Criteria:**
1. Multiple consultants can edit the same project with changes syncing on save (near-real-time)
2. User can view change history for the project
3. User can restore a previous version of the project
4. User can see who last edited each section of the project

---

## Phase 8: Export & Documentation

**Goal:** Users can export all planning artifacts for delivery teams

**Dependencies:** Phase 2, Phase 3, Phase 4 (requires designed components, planned phases, governance setup)

**Requirements:** EXPORT-01, EXPORT-02, EXPORT-03, EXPORT-04

**Success Criteria:**
1. User can export naming convention reference as Word/PDF
2. User can export Dataverse schema diagram as PNG/PDF
3. User can export Power Automate flow map as PNG/PDF
4. User can export Power Apps screen map as PNG/PDF
5. All exports reflect current project state accurately

---

## Progress

| Phase | Status | Requirements | Success Criteria |
|-------|--------|--------------|------------------|
| 1 - Foundation - Project Management | Planned | 9 | 7 |
| 2 - Component Design Core | Pending | 9 | 5 |
| 3 - Planning & Sequencing | Pending | 5 | 4 |
| 4 - Governance & Standards | Pending | 8 | 6 |
| 5 - Templates & AI Assistance | Pending | 7 | 5 |
| 6 - Branding & Visual Presentation | Pending | 7 | 5 |
| 7 - Collaboration | Pending | 6 | 4 |
| 8 - Export & Documentation | Pending | 4 | 5 |

**Total:** 8 phases, 51 v1 requirements, 41 success criteria

---

*Last updated: 2026-02-20*
