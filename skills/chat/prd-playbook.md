---
name: prd-playbook
description: Generate complete, structured PRDs (Product Requirements Documents) for Star Mountain Capital. Use this skill whenever someone asks to write, draft, create, or generate a PRD — even if they just say "write me a PRD for X" or "I need a PRD for this workflow / database / AI agent." Covers three archetypes: (1) Notion Database Schema PRDs, (2) n8n Workflow Automation PRDs, and (3) AI Agent & Product PRDs. Always use this skill before writing any PRD — do not attempt to freehand a PRD without consulting it first.
---

# PRD Playbook — Star Mountain Capital

This skill generates complete, opinionated PRDs for Star Mountain Capital across three archetypes. Read the full prompt template for the matching archetype and fill in the user's inputs before generating.

---

## Step 1: Identify the Archetype

Ask the user (or infer from context) which type of PRD they need:

| # | Archetype | Use When | Examples |
|---|-----------|----------|---------|
| 🗄️ | **Database Schema PRD** | Designing a new Notion database — defining what it tracks, its properties, views, and relations | Operator Bench, Unified Engagements DB, Committees & Teams DB |
| ⚙️ | **n8n Workflow Automation PRD** | Building a new automated workflow — defining trigger, logic steps, data sources, and output | No-Notes Reminder, OPP-01 Calendar Conflict, Friday Reports Automation |
| 🤖 | **AI Agent / Product PRD** | Building an AI agent or multi-module product with personas, phased rollout, and system integrations | Agent Alisha, Borrowing Base ChatBot, K1 Agent Automations |

If unsure, ask: *"Is this a database design, a workflow automation, or an AI agent/product?"*

---

## Step 2: Collect Inputs

For each archetype, collect the required inputs listed in each section below. If the user hasn't provided them all, ask before generating. Use real Star Mountain context when filling examples.

---

## 🗄️ Archetype 1 — Database Schema PRD

**Use when:** Designing a new Notion database.

### Required Inputs
- Database name
- What one row represents
- Primary users
- Key questions this DB must answer (3–5)
- Related databases it should connect to
- Any existing data to migrate
- Known constraints or decisions already made

### Full Generation Prompt

```
You are a senior data architect helping Star Mountain Capital build a new Notion database.
Generate a complete Database Schema PRD using the structure below.
Be specific, opinionated, and lean — every property must earn its column.

--- INPUTS ---
Database name: [e.g. "Deal Pipeline DB"]
What one row represents: [e.g. "One active deal being evaluated for investment"]
Primary users: [e.g. "Investment team, underwriting leads"]
Key questions this DB must answer: [list 3–5 intelligence questions]
Related databases it should connect to: [e.g. "Portfolio Companies, Firm Directory, Interactions"]
Any existing data to migrate: [e.g. "Yes — 47 rows in a legacy Airtable tracker" or "No"]
Known constraints or decisions already made: [e.g. "Must use A/B/C/F grading scale per Jason's requirement"]
--------------

Generate the PRD with these exact sections:

# 1. Executive Summary
One paragraph. What this database is, what it replaces, and the design principle
(e.g. "Minimal architecture — every property must earn its column. Start lean, add as we go.").

# 2. Problem Statement
What questions can't we reliably answer today? List 4–6 specific intelligence gaps
this database will close. Be concrete — use Star Mountain's context
(deal sourcing, portfolio management, LP relationships, etc.).

# 3. Schema Design

## 3a. Unit of Record
One sentence: "One row = one [X]" with 3 concrete examples.

## 3b. Property Table
Generate a full property table with these columns:
- Property name (bold the required/minimum fields)
- Type
- Options or linked DB (for selects and relations)
- Why it exists (one sentence — justify every property)
- Minimum required? (X if it must be filled on intake)

Design principles:
- Bold = minimum required on intake
- Lean toward relations over multi-selects for things that need their own metadata later
- Every date needs a purpose
- Include Created Time and Last Edited Time
- Include a formula for "Days in Current Status" if there's a Status property

## 3c. Select & Multi-Select Option Values
List all options for every select and multi-select property.

## 3d. Formulas & Rollups
For each formula or rollup: property name, where it lives, type, and the logic.

# 4. Intelligence Questions & Views
For each intelligence question, define a view:
- View name, Format, Visible properties, Filters/sorts/groupings

Organize into tiers:
- Tier 1: Strategic
- Tier 2: Coverage / Heatmaps (if applicable)
- Tier 3: Administrative

# 5. Relations Map
Table: This DB Property → Linked Database → Relation Type → Notes
Also list V2+ future relations.

# 6. Status Progression Rules (if applicable)
Table: Status → Entry Criteria → Exit Criteria → SLA

# 7. Migration Plan (if applicable)
- Map old → new properties
- Transformation logic
- Cutover steps (numbered)

# 8. Open Items — Before Build
Numbered list. Owner + status (⏳ Pending / ✅ Resolved).

# 9. Open Items — Post-Build / V2+
Out of scope for V1.
```

### Tips
- Be specific with intelligence questions. Bad: *"Show me all deals."* Good: *"Which deals have been in diligence for more than 2 weeks with no owner action?"*
- List ALL related databases even if the relation isn't fully defined yet.
- Name constraints upfront (grading scales, field naming conventions, data sources to link).
- After generation, review the property table: *"Would I actually fill this field in on every new row?"* If no, cut it.

---

## ⚙️ Archetype 2 — n8n Workflow Automation PRD

**Use when:** Building a new automated workflow in n8n.

### Required Inputs
- Workflow name
- Phase & Priority (e.g. Phase 2 · P1)
- Target completion date
- Effort / Impact (Low / Medium / High)
- What this replaces
- Trigger type + frequency/schedule
- Data sources involved
- Core logic summary (2–4 sentences)
- Output / delivery method + recipients
- Known constraints (especially n8n limitations)
- Open items already known

### Full Generation Prompt

```
You are a senior automation architect helping Star Mountain Capital build a new n8n workflow.
Generate a complete n8n Workflow Automation PRD using the structure below.
Be precise about trigger mechanics, logic branching, and API specifics.

--- INPUTS ---
Workflow name: [e.g. "OPP-01 — Calendar Conflict Detection"]
Phase & Priority: [e.g. "Phase 2 · P1"]
Target completion: [e.g. "End of June 2026"]
Effort estimate: [Low / Medium / High]
Impact: [Low / Medium / High]
What this replaces: [e.g. "Manual conflict checking during daily rundown prep"]
Trigger type: [Schedule (with frequency) / Webhook / Event / Manual]
Data sources involved: [list all]
Core logic summary: [2–4 sentences]
Output / delivery: [e.g. "Batched HTML email via Microsoft Graph sendMail"]
Recipients or destinations: [who/where]
Known constraints: [e.g. "n8n native Notion node cannot read page block content — must use HTTP Request node"]
Open items you already know about: [list any]
--------------

[Header block — inline text, not a heading]
Phase: [X] · Priority: [P0/P1/P2]
Target: [date]
Effort: [Low/Medium/High] · Impact: [Low/Medium/High]
Replaces: [what it replaces]

## Objective
One paragraph. What the workflow does, why it matters, what problem it eliminates. No bullet points.

## User Stories
3–4 stories: "As [role], I want [capability] so that [outcome]."
Use real Star Mountain roles where applicable.

## Trigger
- Type (Schedule / Webhook / Event)
- If schedule: exact time, days, timezone
- If delta/poll: polling interval and state management (deltaToken, static data, etc.)
- Technical notes for n8n configuration

## Persistent Logic (if applicable)
Describe deduplication, re-alert windows, or stateful behavior. Include where state is stored.

## Data Sources
Table: Source · What it provides · Access method (n8n native / HTTP Request / Graph API / etc.)

## Core Logic
Numbered steps. For each:
- What it does
- n8n node type
- Technical detail (API endpoint, filter logic, field mapping)
- Branching conditions
Flag wherever n8n native node has limitations.

## n8n Architecture Sketch
Code block (plain text), full workflow as linear/branching flow:

Schedule Trigger
  → [Step]
  → IF [condition] → stop
  → Loop: [describe]
      → [substep]
  → [Output step]

## Output / Delivery Format
- Output type, sender, recipients
- Subject line format
- Per-record fields in output
- Conditional sending logic
- Formatting notes (HTML, inline CSS, etc.)

## Assumptions
Bulleted list. Format: "[System/field] [assumption]"

## Open Items
Table: # · Item · Owner · Status (⏳ Pending / ✅ Resolved)
```

### Tips
- Be specific about the trigger — include exact time, timezone, and days.
- Name the n8n credential if known (e.g., `TearSheets Notion credential`).
- List known n8n limitations upfront (e.g., Notion native node can't read block content).
- If deduplication is needed, mention it explicitly.

---

## 🤖 Archetype 3 — AI Agent & Product PRD

**Use when:** Building a multi-module AI agent or complex product with personas, phased rollout, and integrations.

### Required Inputs
- Product / Agent name + codename
- One-line description
- Primary users / personas (2–4 people with roles)
- End beneficiary
- Core problem being solved
- Features / modules to include
- Systems to integrate with (Notion, n8n, V7, Claude, Meridian, Microsoft 365, etc.)
- Phase 1 target date
- Human process being replaced or sunset (if any)
- Known constraints
- Version / stage

### Full Generation Prompt

```
You are a senior product manager helping Star Mountain Capital write a PRD
for a new AI agent or product. Generate a complete AI Agent / Product PRD
using the structure below. Be specific about user personas, module priorities,
integrations, and rollout sequencing.

--- INPUTS ---
Product / Agent name: [e.g. "Agent Alisha"]
Project codename (if any): [e.g. "Project Alisha"]
One-line description: [e.g. "An AI operations agent that manages calendar, travel, CRM, and protocol compliance"]
Primary users / personas: [2–4 people with roles]
End beneficiary: [e.g. "Principal — Managing Partner"]
Core problem being solved: [2–4 sentences]
Features / modules to include: [list capabilities]
Systems to integrate with: [list all — Notion, n8n, Claude, Microsoft 365, V7, Meridian, etc.]
Phase 1 target date: [e.g. "End of Q3 2026"]
Is there a human process being replaced or sunset?: [Yes/No — describe if yes]
Known constraints: [e.g. "Must have human approval before any outbound communication"]
Version / stage: [e.g. "1.0 MVP"]
--------------

[Header block]
Project [Codename] | Star Mountain Capital | [Month Year]
Version: [X.X]
Target Launch: [date or milestone]

## 1. Product Vision
One paragraph: what it is, who it serves, what it does, how it fits SMC's automation strategy.
Include: admin/power user, human dependency reduced, long-term evolution path.

## 2. Problem Statement
4–6 numbered concrete problems. Name the person, failure mode, frequency/impact.
End with: "Questions we can't reliably answer today:" + 3–5 gaps.

## 3. Users & Personas
For each persona:
- **Name — Role / Title**
- Needs, Interaction Model, Key Requirement (first-person quote)
Include End Beneficiary if applicable.

## 4. MVP Feature Set (v1.0)
Modules with priority. For each:

### Module [N]: [Name] (P0 / P1 / P2)
**Purpose:** One sentence.
Table: Feature · Description · Automation Level
Automation Levels: Full / Hybrid / Full detection human review / Full detection human action
**Replaces:** [human task eliminated]

## 5. System Architecture

### Integration Points
Table: System · Integration Type · Purpose

### Design Principles
4–6 non-negotiables (e.g., Human-in-the-loop, Audit-first, FinOps admin approval).

### Data Flow
Code block (plain text):

[Source 1] ──┐
              ├──► [Agent Core] ──┬──► [Output 1]
[Source 2] ───┘                   └──► [Output 2]

## 6. Rollout Plan
3 phases. Each:
### Phase [N] — [Name] ([Months])
**Goal:** One sentence.
Table: Module · Target Date · Deliverables
**Exit Criteria:** 2–3 measurable conditions.

## 7. Success Metrics
Table: Metric · Baseline · Target · Measurement Method
8–10 metrics mixing operational, quality, and business outcome.

## 8. Risks & Mitigations
Table: Risk · Likelihood · Impact · Mitigation (4–6 risks)

## 9. Dependencies & Prerequisites
Table: Dependency · Owner · Status · Required By (Phase)

## 10. Open Questions
5–8 numbered questions. Categorize: Architecture / Scope / Stakeholder decisions.

## 11. Appendices
### A. Source Materials
### B. Related Documents
### C. Action Items — format: "[Owner]: [Action]"
```

### Tips
- Use P0/P1/P2 rigorously — no more than 3 P0 modules.
- Name the human process being replaced explicitly.
- Exit criteria must be measurable, not date-driven.
- The Data Flow diagram is the most useful artifact for the builder.
- Every module needs an admin control point.

---

## General Principles (All Archetypes)

- **Be opinionated.** Make recommendations; flag disagreements as open items.
- **Use real context.** Reference SMC systems (Meridian, Notion, n8n, V7, Microsoft 365, Claude) and actual workflows.
- **Flag blockers explicitly.** Pre-build blockers → "Before Build." Everything else → "Post-Build / V2+."
- **Output as markdown.** Well-structured, not wrapped in code blocks unless asked.
- **Every section earns its place.** If a section doesn't apply, omit it.
- **SMC vocabulary throughout:** LP, GP, IC, LPA, BDC, unitranche, NAV, PCAP, SOI, MOIC, DPI.
