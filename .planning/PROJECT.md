# Project: Power Platform Roadmap Planner

**Initialized:** 2026-02-19
**Status:** Planning

---

## What This Is

A planning and delivery tool for Power Platform consultants and implementers. Helps teams scope, sequence, design, and present Power Platform implementations — from pre-sales discovery through live delivery — consistently and without re-inventing the wheel on every engagement.

The tool is used throughout the entire client lifecycle: create a proposal roadmap pre-sales, refine it at kickoff, track progress during delivery. Multiple consultants collaborate on the same project.

---

## Core Value

**Eliminate the blank-page problem for every new Power Platform project.**

Without this tool, consultants start each engagement from scratch — different scoping approaches, inconsistent client communication, no standard for what to build first. The result is uneven quality, slow pre-sales, and delivery mistakes (wrong order, missing dependencies, licensing surprises).

With this tool, a consultant opens a new project, picks from proven templates, gets AI-assisted guidance on what to build and in what order, and walks into the client meeting with a professional visual roadmap and a technical delivery plan — in an hour instead of a day.

---

## Target Users

**Primary:** Power Platform consultants and solution architects
- Plan and deliver Power Platform projects for clients
- Need to produce consistent, professional roadmaps fast
- Work in teams — multiple people on one project
- Often non-technical stakeholders in the room during presentations

**Future:** Other consultancies (SaaS potential)
- Start as an internal tool, validate, then potentially offer to the market

---

## Problem Space

Four pain points driving this build:

| Pain Point | Impact |
|-----------|--------|
| No standard way to scope | Every consultant does it differently — inconsistent quality, variable estimates |
| Hard to explain to clients | Technical plans confuse stakeholders — roadmaps need to be visual and simple |
| No dependency tracking | Things get built in the wrong order (e.g. Power App before Dataverse schema) |
| No reusable templates | Common scenarios (approval flows, form replacements) get re-scoped from scratch |

---

## Key Capabilities

### Core Planning
- **Sequencing engine** — tells consultants what to establish first, what depends on what
- **Template library** — common Power Platform scenarios pre-built, fully customizable
- **AI-assisted suggestions** — based on project inputs, suggest phases, components, and sequencing
- **Manual input** — consultants can override or build from scratch

### Data & Architecture Design
- **Dataverse / SharePoint List designer** — define tables, columns, field types, relationships visually
- **Power Apps screen planner** — map out required screens and navigation flows
- **Power Automate flow mapper** — diagram which flows are needed, their triggers, and what they connect to

### Governance & Standards
- **Connector + licensing tracker** — as components are added, flags when Premium licensing is triggered
- **Environment strategy planner** — define Dev/Test/Prod structure, promotion paths, ALM approach
- **Naming convention generator** — consistent names across apps, flows, tables; exportable as reference doc

### Presentation & Branding
- **Color/brand theming** — pre-made palettes, fully customizable; applied consistently across all outputs
- **Visual stakeholder roadmap** — high-level phases/timeline suitable for client presentations
- **Detailed delivery breakdown** — technical plan for the delivery team

### Collaboration
- **Multi-consultant editing** — multiple team members on one project simultaneously
- **Project history** — changes tracked, previous versions accessible

---

## Output Types

1. **Visual roadmap** — stakeholder-facing, phase/timeline view, exportable as PDF
2. **Technical delivery plan** — detailed component list with dependencies, sequencing, and owners
3. **Architecture documentation** — Dataverse schema, screen maps, flow diagrams
4. **Naming convention reference** — exported doc of all naming standards for the project

---

## Key Decisions

| Decision | Options | Status |
|----------|---------|--------|
| Tech stack | Web app (React/Next.js) vs. built IN Power Platform | Pending — see note below |
| AI provider | OpenAI GPT-4 vs. Azure OpenAI vs. Copilot Studio | Pending |
| Auth | Microsoft/Entra ID (M365 SSO) vs. email/password | Pending |
| Data storage | Supabase/PostgreSQL vs. Dataverse vs. SQLite | Pending |
| Deployment | Internal hosted vs. SaaS (multi-tenant) | Start internal, decide post-MVP |

**Tech stack note:** Building as a **web app (React/Next.js)** is recommended over a Power Apps canvas/model-driven app because: (1) the UI requires complex interactions — drag-and-drop sequencing, visual diagrams, color pickers; (2) AI integration is straightforward; (3) SaaS deployment is natural. The tool can still export to and integrate WITH Power Platform regardless of how it's built.

---

## Scope Boundaries

### v1 — Internal tool, core planning workflow
Scope decisions made during initialization.

### v2 — SaaS + integrations
- Export to Azure DevOps (work items from roadmap)
- Export to Microsoft Planner
- Connect to client's SharePoint/Dataverse to read existing structure
- Effort estimator (t-shirt sizing per component)
- Client portal (read-only view for clients)

### Out of Scope (permanently)
- Actual Power Platform development (this plans it, doesn't build it)
- Real-time collaboration (Google Docs-style live cursors) — too complex for v1
- Automated billing/invoicing — different product

---

## Requirements

### Validated
*(None yet — ship to validate)*

### Active
*(Defined in REQUIREMENTS.md)*

### Out of Scope
- Real-time co-editing with live cursors — v2 at earliest
- Power Platform development environment — this is a planning tool, not an IDE
- Client billing features — separate product

---

## Key Decisions Log

*(Populated as decisions are made during development)*

---

*Last updated: 2026-02-19 after initialization*
