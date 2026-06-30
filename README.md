# SMC Claude Skills Library

**Star Mountain Capital — Curated AI Skills for the Investment & Operations Teams**

---

## What Is This?

This repository contains **Claude Skills** — pre-built instruction sets that teach Claude how to work specifically for Star Mountain Capital. Instead of explaining your role, the firm's terminology, and what you need every single time you open Claude, a Skill loads all of that context automatically. You just start asking questions.

Think of a Skill like a briefing document that Claude reads before you walk into the room. It knows you're at a PE/credit firm, knows what an LPA is, knows how SMC structures deals, and knows the difference between a platform and an add-on. You don't have to explain any of it.

**Without a Skill:**
> *"I'm an IR professional at a private equity firm. We do lower middle market buyouts. I need to write a quarterly letter to our limited partners about..."*

**With the `lp-memo-writer` Skill loaded:**
> *"Draft a Q2 letter for SMC Credit Fund III. NAV is $387M, net investment income was $12M, and we closed one new deal."*

---

## What Is GitHub?

GitHub is a website where files are stored, organized, and shared — think of it like Dropbox or SharePoint, but designed for files that need a full version history (every change ever made is tracked and reversible). You don't need to understand how GitHub works to use these Skills. The only thing you need is the link to a specific file. From there, installing a Skill is as simple as copying a URL or pasting text into Claude. This repository lives in the Star Mountain Capital GitHub organization so that the IR, FinOps, and investment teams all share the same up-to-date set of Skills — and when the Skills are improved, everyone gets the update automatically.

---

## Two Ways to Use Claude at SMC

| Tool | Who Uses It | Best For |
|------|-------------|----------|
| **claude.ai** (browser) | All team members | Writing, research, analysis, Q&A, document review |
| **Claude Code** (terminal) | Technical team / FinOps | Automation, scripts, data processing, building tools |

Skills in this library are organized to match:
- `/skills/chat/` — for **claude.ai** users (no coding required)
- `/skills/code/` — for **Claude Code** users (technical / FinOps)
- `/skills/shared/` — work in **both** contexts

---

## How to Install a Skill

### Option A: claude.ai Users — Copy & Paste (No Technical Knowledge Needed)

1. **Find the Skill you want** in the table below. Each skill has a file path like `skills/chat/lp-memo-writer.md`.

2. **Open the raw file on GitHub:**
   - Go to [https://github.com/Star-Mountain-Capital/Claude-Skills](https://github.com/Star-Mountain-Capital/Claude-Skills)
   - Navigate to the skill file (e.g., click `skills` → `chat` → `lp-memo-writer.md`)
   - Click the **"Raw"** button in the top-right corner of the file view
   - You'll see plain text. Press `Ctrl+A` (Windows) or `Cmd+A` (Mac) to select all, then `Ctrl+C` / `Cmd+C` to copy it.

3. **Install in claude.ai:**
   - Go to [claude.ai](https://claude.ai) and sign in
   - Click your profile icon → **"Profile & Settings"** → **"Skills"**
   - Click **"Add Skill"** or **"Create Skill"**
   - Paste the text you copied into the skill editor
   - Give it a name (use the skill name from the table below) and save

4. **Use it:** Start a new conversation. Skills can be toggled on or off per conversation from the conversation settings panel.

> **Tip:** You only need to install a skill once. After that it's saved to your account and available in every conversation.

---

### Option B: Claude Code Users — Terminal Install

**Install a single skill:**
```bash
mkdir -p ~/.claude/skills
curl -o ~/.claude/skills/lp-memo-writer.md \
  https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/lp-memo-writer.md
```

**Install all SMC skills at once:**
```bash
git clone https://github.com/Star-Mountain-Capital/Claude-Skills.git ~/smc-claude-skills
mkdir -p ~/.claude/skills
cp ~/smc-claude-skills/skills/chat/*.md ~/.claude/skills/
cp ~/smc-claude-skills/skills/code/*.md ~/.claude/skills/
cp ~/smc-claude-skills/skills/shared/*.md ~/.claude/skills/
echo "All SMC skills installed."
```

**Install for a specific project only:**
```bash
mkdir -p .claude/skills
cp ~/smc-claude-skills/skills/code/finops-reconciliation.md .claude/skills/
```

**Keep skills up to date:**
```bash
cd ~/smc-claude-skills && git pull && cp skills/**/*.md ~/.claude/skills/
```

---

## Skill Library

### Chat Skills — for claude.ai Users

No coding required. Paste documents, describe your situation, and Claude responds with SMC-context-aware outputs.

| Skill Name | Audience | What It Does | Install Command |
|-----------|----------|-------------|-----------------|
| **LP Memo Writer** | IR / Investor Relations | Drafts quarterly LP letters, capital call notices, distribution notices, and investor updates. Knows SMC fund structures, LP typology, and institutional tone standards. | `curl -o ~/.claude/skills/lp-memo-writer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/lp-memo-writer.md` |
| **Meeting Intelligence** | All investment & ops | Processes meeting transcripts and notes from IC meetings, LP calls, and portfolio reviews. Extracts action items, decisions, deal developments, and follow-ups. | `curl -o ~/.claude/skills/meeting-intelligence.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/meeting-intelligence.md` |
| **PE Analyst** | Investment team | Acts as a senior PE analyst. Screens deals, builds investment theses, prioritizes diligence, frames valuations, and helps draft IC materials for LMM buyouts and credit deals. | `curl -o ~/.claude/skills/pe-analyst.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/pe-analyst.md` |
| **Deal Memo Writer** | Investment team | Writes and edits Investment Committee memos and credit memos in full PE equity or private credit format. Understands SMC's memo standards throughout. | `curl -o ~/.claude/skills/deal-memo-writer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/deal-memo-writer.md` |
| **LP Relations** | IR / Investor Relations | Handles LP relations workflows — DDQ responses, LP query drafts, meeting prep and briefings, and communication strategy for institutional investors. | `curl -o ~/.claude/skills/lp-relations.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/lp-relations.md` |
| **Executive Comms** | Leadership / all teams | Drafts internal executive communications — firm-wide announcements, leadership updates, policy memos, board briefings, and sensitive communications like departures. | `curl -o ~/.claude/skills/executive-comms.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/executive-comms.md` |
| **Read PDF** | All teams | Reads, extracts, and analyzes PDF documents — CIMs, LPAs, credit agreements, fund reports, regulatory filings. Provides structured summaries, pulls financial tables, and flags risks. Works with file attachments in claude.ai and pdfplumber/pypdf code patterns in Claude Code. | `curl -o ~/.claude/skills/readpdf.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/readpdf.md` |
| **Picture Generation** | All teams | Generates professional charts, diagrams, org charts, and presentation visuals for SMC use cases — fund performance charts, portfolio composition graphics, process flows, deal funnel maps. Follows SMC brand standards (Navy/Gold palette) for LP-facing and IC materials. | `curl -o ~/.claude/skills/picture-generation.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/picture-generation.md` |
| **PRD Playbook** | Ops / FinOps / builders | Generates complete PRDs for three SMC archetypes: Notion Database Schema PRDs, n8n Workflow Automation PRDs, and AI Agent/Product PRDs. Includes full generation prompts and structured templates for each archetype. | `curl -o ~/.claude/skills/prd-playbook.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/prd-playbook.md` |
| **Skill Creator** | All teams | Meta-skill for creating new Claude Skills for the SMC library. Guides the user through scoping, inputs, outputs, and SMC context questions, then generates a complete, properly formatted SKILL.md ready to push to GitHub. | `curl -o ~/.claude/skills/skill-creator.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/skill-creator.md` |
| **Email Triage** | All teams | Triages and prioritizes inbox snapshots, drafts responses to LP inquiries, deal counterparty emails, portfolio company requests, and internal correspondence. Flags items requiring senior attention or legal/compliance review. Institutional tone throughout. | `curl -o ~/.claude/skills/email-triage.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/email-triage.md` |
| **Presentation Builder** | Investment / IR / ops | Builds structured slide narratives and presentation outlines for IC presentations, LP update decks, board briefings, annual meetings, and team updates. Outputs slide-by-slide narrative frameworks with talking points. Executive-grade, investor-ready tone. | `curl -o ~/.claude/skills/presentation-builder.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/presentation-builder.md` |

---

### Code Skills — for Claude Code / Technical Users

Designed for use with Claude Code in the terminal. Includes code patterns, automation frameworks, and data processing pipelines built around SMC's FinOps and investment technology stack.

| Skill Name | Audience | What It Does | Install Command |
|-----------|----------|-------------|-----------------|
| **FinOps Reconciliation** | FinOps team | Builds GL vs. administrator reconciliation scripts, management fee calculators, capital account roll-forwards, and month-end close automation. Adapted from Anthropic's gl-reconciler + month-end-closer cookbooks. | `curl -o ~/.claude/skills/finops-reconciliation.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/finops-reconciliation.md` |
| **Data Parser** | FinOps / investment | Parses and normalizes financial data files — Excel workbooks, PDF statements, Word documents, and CSVs — into clean structured formats for analysis or downstream processing. | `curl -o ~/.claude/skills/data-parser.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/data-parser.md` |
| **Notion API Patterns** | FinOps / Meridian team | Provides SMC-specific Notion API patterns for the Meridian FinOps Hub — querying, creating, and updating Notion databases for deal tracking, fund reporting, period status, and LP data. | `curl -o ~/.claude/skills/notion-api-patterns.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/notion-api-patterns.md` |
| **Valuation Reviewer** | FinOps / investment | Ingests and validates GP valuation packages, checks methodologies and implied multiples, builds NAV summaries, and generates LP reporting packs. Adapted from Anthropic's valuation-reviewer cookbook. | `curl -o ~/.claude/skills/valuation-reviewer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/valuation-reviewer.md` |
| **Investment Research** | Investment team | Builds investment research pipelines — sector screeners, comparable company tables, earnings update processors, and research note generators. Adapted from Anthropic's market-researcher + earnings-reviewer cookbooks. | `curl -o ~/.claude/skills/investment-research.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/investment-research.md` |
| **LP Statement Auditor** | FinOps / IR | Audits LP capital account statements and distribution notices before distribution — validates math, cross-checks fund records, and generates sign-off reports for IR and CFO review. Adapted from Anthropic's statement-auditor cookbook. | `curl -o ~/.claude/skills/lp-statement-auditor.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/lp-statement-auditor.md` |

---

### Shared Skills — Work in Both claude.ai and Claude Code

| Skill Name | Audience | What It Does | Install Command |
|-----------|----------|-------------|-----------------|
| **Financial Analysis** | Investment / FinOps / all | Performs institutional-quality financial analysis — ratio analysis, DCF valuation, EBITDA addback review, budget variance analysis, and LBO return framing. Adapted from alirezarezvani's financial-analyst skill. | `curl -o ~/.claude/skills/financial-analysis.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/financial-analysis.md` |
| **Document Summarizer** | All teams | Summarizes complex financial and legal documents — CIMs, LPAs, credit agreements, PPMs, regulatory filings — with SMC-relevant callouts and structured output templates. | `curl -o ~/.claude/skills/document-summarizer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/document-summarizer.md` |
| **Deal Screening** | Investment team | Rapidly screens investment opportunities against SMC's mandate — evaluating fit, valuation context, business quality, return potential, and key diligence questions for PE equity and private credit. | `curl -o ~/.claude/skills/deal-screening.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/deal-screening.md` |
| **Compliance Monitor** | Compliance / legal / IR | Supports compliance workflows — KYC/AML onboarding reviews, MNPI flagging, regulatory filing deadline tracking, and LP communication disclaimer reviews. All outputs require compliance officer review. | `curl -o ~/.claude/skills/compliance-monitor.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/compliance-monitor.md` |

---

## Skill Packs by Team

Not sure which skills to install? Start with the pack for your team:

### Investment Team
```bash
curl -o ~/.claude/skills/pe-analyst.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/pe-analyst.md
curl -o ~/.claude/skills/deal-memo-writer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/deal-memo-writer.md
curl -o ~/.claude/skills/deal-screening.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/deal-screening.md
curl -o ~/.claude/skills/financial-analysis.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/financial-analysis.md
curl -o ~/.claude/skills/document-summarizer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/document-summarizer.md
```

### Investor Relations Team
```bash
curl -o ~/.claude/skills/lp-memo-writer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/lp-memo-writer.md
curl -o ~/.claude/skills/lp-relations.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/lp-relations.md
curl -o ~/.claude/skills/meeting-intelligence.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/meeting-intelligence.md
curl -o ~/.claude/skills/document-summarizer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/document-summarizer.md
```

### FinOps Team
```bash
curl -o ~/.claude/skills/finops-reconciliation.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/finops-reconciliation.md
curl -o ~/.claude/skills/data-parser.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/data-parser.md
curl -o ~/.claude/skills/notion-api-patterns.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/notion-api-patterns.md
curl -o ~/.claude/skills/lp-statement-auditor.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/lp-statement-auditor.md
curl -o ~/.claude/skills/valuation-reviewer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/valuation-reviewer.md
```

### Legal & Compliance Team
```bash
curl -o ~/.claude/skills/compliance-monitor.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/compliance-monitor.md
curl -o ~/.claude/skills/document-summarizer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/document-summarizer.md
curl -o ~/.claude/skills/executive-comms.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/executive-comms.md
```

### Everyone Pack — Start Here
Universal skills useful for every team member, regardless of role.
```bash
curl -o ~/.claude/skills/readpdf.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/readpdf.md
curl -o ~/.claude/skills/picture-generation.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/picture-generation.md
curl -o ~/.claude/skills/document-summarizer.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/shared/document-summarizer.md
curl -o ~/.claude/skills/meeting-intelligence.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/meeting-intelligence.md
curl -o ~/.claude/skills/executive-comms.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/executive-comms.md
```

### Builder Pack — Ops, FinOps, and Technical Team
For team members building workflows, databases, AI agents, or new Claude Skills.
```bash
curl -o ~/.claude/skills/prd-playbook.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/prd-playbook.md
curl -o ~/.claude/skills/skill-creator.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/chat/skill-creator.md
curl -o ~/.claude/skills/notion-api-patterns.md https://raw.githubusercontent.com/Star-Mountain-Capital/Claude-Skills/main/skills/code/notion-api-patterns.md
```

---

## Sources & Credits

These skills were curated from public repositories and adapted for SMC's private equity and private credit context:

| Source Repository | Skills Adapted |
|------------------|----------------|
| [anthropics/skills](https://github.com/anthropics/skills) | `internal-comms` → executive-comms; `xlsx` + `pdf` → data-parser |
| [anthropics/financial-services](https://github.com/anthropics/financial-services) | `gl-reconciler` + `month-end-closer` → finops-reconciliation; `valuation-reviewer` → valuation-reviewer; `statement-auditor` → lp-statement-auditor; `market-researcher` + `earnings-reviewer` → investment-research; `pitch-agent` → pe-analyst; `meeting-prep-agent` → meeting-intelligence; `kyc-screener` → compliance-monitor |
| [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) | `finance/skills/financial-analyst` → financial-analysis |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | `meeting-insights-analyzer` → meeting-intelligence |
| SMC original | `lp-memo-writer`, `deal-memo-writer`, `lp-relations`, `deal-screening`, `notion-api-patterns` |

---

## Maintenance & Contributing

This library is maintained by the Casey Raffone. To suggest a new skill or report an issue:

1. Open an issue in this GitHub repository, or
2. Contact the BizOps team directly

**Skill format standard:**
```markdown
---
name: kebab-case-name
description: Full description of when to use this skill and what it does.
---

# Skill Title

[Skill instructions]
```

**Guidelines for new skills:**
- Reference SMC's lower middle market PE/credit context
- Use SMC terminology (LP, GP, LPA, BDC, unitranche, platform, add-on, etc.)
- Reflect institutional tone standards
- Never include PII, confidential LP data, or portfolio company non-public information

---

*Maintained by the SMC BizOps Team*  
*Repository: [https://github.com/Star-Mountain-Capital/Claude-Skills](https://github.com/Star-Mountain-Capital/Claude-Skills)*
