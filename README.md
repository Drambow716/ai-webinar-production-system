# AI Webinar Production System

An internal AI-powered platform that automates end-to-end webinar content production for a B2B SaaS company running **30+ webinars per month**.

Built to eliminate **150–300 hours/month** of manual copywriting while maintaining brand consistency and full human editorial control.

> ⚠️ This is a proprietary internal system. Code is confidential. This repository documents the architecture, design decisions, and outcomes for portfolio purposes.

---

## The Problem

A B2B SaaS company running 30+ webinars per month was spending 5–10 hours per event on manual content production — writing landing page copy, email sequences, slide narratives, and messaging frameworks from scratch each time.

The process was:
- **Slow** — each webinar required hours of copywriting across multiple formats
- **Inconsistent** — brand voice and messaging varied across team members
- **Disconnected** — no institutional memory of what messaging worked before

---

## The Solution

An end-to-end AI pipeline that takes a webinar brief as input and produces a complete, brand-aligned content package in minutes — with humans in control of review and final approval.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     INPUT LAYER                         │
│                                                         │
│   Airtable Record (webinar brief)                       │
│   - Title, product, audience, topic, regions            │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│               ORCHESTRATION LAYER (n8n)                 │
│                                                         │
│   Webhook trigger → workflow execution                  │
│   Resume tokens for async human-in-the-loop pauses      │
│   Status management across pipeline stages              │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│               AI CONTENT ENGINE                         │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Multi-Step Generation Pipeline (GPT-4o)        │   │
│  │                                                 │   │
│  │  Step 1: Messaging Framework                    │   │
│  │    → persona, pain points, resolution angle     │   │
│  │                                                 │   │
│  │  Step 2: Learning Outcomes & Capabilities       │   │
│  │    → 5 outcomes + 5 product capabilities        │   │
│  │                                                 │   │
│  │  Step 3: Demo Narrative                         │   │
│  │    → focus, chapter 1, chapter 2, chapter 3     │   │
│  │                                                 │   │
│  │  Step 4: Landing Page Abstract                  │   │
│  │    → title, subtitle, paragraphs, bullets       │   │
│  │                                                 │   │
│  │  Step 5: Email Sequences (EM1–EM3)              │   │
│  │    → subject, preheader, header, body           │   │
│  │                                                 │   │
│  │  Step 6: PowerPoint Slide Deck                  │   │
│  │    → AI-generated slide content                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────┐  ┌──────────────────────┐    │
│  │   RAG Knowledge Base │  │   Build History      │    │
│  │                      │  │                      │    │
│  │  Semantic search via │  │  Past approved       │    │
│  │  embeddings at each  │  │  outputs inform      │    │
│  │  generation step     │  │  future generations  │    │
│  │                      │  │                      │    │
│  │  Categories:         │  │  Matched by:         │    │
│  │  - Brand guidelines  │  │  - Product code      │    │
│  │  - Product briefs    │  │  - Audience segment  │    │
│  │  - Competitor intel  │  │                      │    │
│  │  - Past campaigns    │  │  Self-improving loop │    │
│  └──────────────────────┘  └──────────────────────┘    │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│           REVIEW DASHBOARD (Next.js Web App)            │
│                                                         │
│  Status: GENERATING → PENDING → APPROVED → DONE         │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Review Interface                               │   │
│  │  - Side-by-side draft view                      │   │
│  │  - Field-level AI rewrite on demand             │   │
│  │    ("make this more urgent for CFOs")           │   │
│  │  - Full abstract or full email rewrite          │   │
│  │  - One-click approve                            │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Settings & Configuration (no-code tuning)      │   │
│  │  - Model selection (GPT-4o, GPT-4o-mini, etc.)  │   │
│  │  - Temperature per use case                     │   │
│  │  - All system prompts editable via UI           │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Knowledge Base Manager                         │   │
│  │  - Add/edit brand guidelines, product content   │   │
│  │  - Auto-embeds on save (text-embedding-3-small) │   │
│  │  - Instantly available to all future generations│   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────┬───────────────────────────────────┘
                      │ (on approval)
                      ▼
┌─────────────────────────────────────────────────────────┐
│           POST-APPROVAL AUTOMATION (n8n)                │
│                                                         │
│   Resume token triggers post-approval workflow          │
│   → Format and upload landing page copy doc             │
│   → Format and upload email sequence doc                │
│   → Generate and upload PowerPoint deck                 │
│   → Deliver all assets to Google Drive                  │
└─────────────────────────────────────────────────────────┘
```

---

## Key Features

### Multi-Step Generation Pipeline
Content is generated in a structured sequence — messaging first, then outcomes, then demo narrative, then emails — each step informed by the previous. This produces a coherent, internally consistent content package rather than disconnected outputs.

### RAG Knowledge Base
A curated knowledge base is searched semantically at generation time using vector embeddings. Relevant brand context, product details, and competitive positioning are automatically injected into every prompt — no manual context needed per webinar.

### Build History & Self-Improvement
Every approved output is stored with metadata (product, audience segment). Future generations for the same product/audience combination are informed by what was previously approved. The system improves over time without retraining.

### Human-in-the-Loop Review
Reviewers interact with AI drafts through a structured dashboard. Any individual field (title, email body, bullet list) or full section (entire abstract, entire email) can be rewritten with a natural language instruction. Full human control before anything goes live.

### Resume Token Workflow
n8n workflows pause after generating drafts and resume automatically once a reviewer approves — no manual handoffs, no dropped tasks, full audit trail of status transitions.

### No-Code AI Configuration
All system prompts, model selection, and temperature settings are stored in the database and editable through a settings UI. The marketing team can tune AI behavior without touching code or requiring engineering support.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14, TypeScript, React |
| Backend | Next.js API Routes, Prisma ORM |
| Database | SQLite (via Prisma) |
| AI Generation | OpenAI GPT-4o |
| AI Embeddings | OpenAI text-embedding-3-small |
| Automation | n8n (self-hosted) |
| Data Source | Airtable |
| Asset Delivery | Google Drive API |
| Auth | Middleware-based session authentication |

---

## Results

| Metric | Result |
|---|---|
| Webinars served per month | 30+ |
| Manual content hours eliminated | 150–300 hrs/month |
| Content formats automated | 6 (messaging, outcomes, demo, abstract, emails, slides) |
| Human control maintained | Full review + rewrite + approval workflow |
| Time to first draft | Minutes (vs. hours manually) |

---

## Design Decisions

### Why RAG over fine-tuning?
Brand knowledge, product details, and campaign history change frequently. RAG allows real-time updates — a content manager adds a document and it's immediately available to all future generations. Fine-tuning would require retraining every time content changes.

### Why human-in-the-loop?
B2B SaaS marketing copy for a named, senior audience (CFOs, VPs of Finance) requires editorial judgment. The system eliminates 80%+ of the work but keeps marketers as final decision-makers — ensuring quality, accountability, and the subtle judgment calls that pure automation misses.

### Why n8n for orchestration?
Non-technical stakeholders need visibility into pipeline status and the ability to debug stuck workflows. n8n's visual interface lets the marketing ops team monitor and troubleshoot without engineering support. The resume token pattern enables clean async handoffs between the AI layer and the human review step.

### Why a custom web app over a no-code tool?
The review experience needed to be tailored — showing AI-generated fields in context, supporting field-level rewrites with natural language, and managing multi-status workflow states. No off-the-shelf tool could provide this UX without significant compromise.

---

## Related Work

I'm building additional automation systems at insightsoftware, including:
- Automated sales asset generation pipelines
- AI-assisted PowerPoint deck builders
- Content approval and distribution workflows

---

## About

I design and build AI-powered systems for revenue and marketing teams — combining deep B2B SaaS experience with hands-on technical ability to build systems that actually move business metrics.

[Upwork Profile](https://www.upwork.com/freelancers/walidlakouader) · [LinkedIn](https://www.linkedin.com/in/walid-lakouader-001/)
