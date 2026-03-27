# Hucki - Lightweight IRCA RAG Engine

**Status:** Active Development
**Start Date:** 2026-03-27
**Core Value:** Zero-cost, Claude Code-native RAG for automated research synthesis

---

## What This Is

Hucki is a headless RAG (Retrieval-Augmented Generation) engine purpose-built for the IRCA workflow (Ideate → Research → Context → Action). It takes research files, embeds them in Convex, and uses Claude Code CLI to synthesize actionable context documents—enabling fully automated research-to-execution pipelines via n8n.

**Not a chat interface. Not a UI. A programmatic RAG service.**

## The Problem

Existing RAG solutions have critical gaps for automation workflows:

- **NotebookLM:** Requires Google credentials (security risk), no API, manual upload
- **AnythingLLM:** Uses Claude API ($$$), heavyweight (50k LOC), UI-first design
- **Custom solutions:** Either use expensive APIs or require complex vector DB setup

**What's missing:** A lightweight, API-driven RAG that uses Claude Code (already subscribed) with project-siloed storage in Convex (already deployed).

## Core Value

**The ONE thing that must work:**
Generate high-quality, actionable context from research files using Claude Code CLI—with zero external API costs and complete data privacy.

If context synthesis is poor, nothing else matters.

## Context

### User (Primary)
**Billi** (AI agent) - Orchestrates IRCA workflows, commissions builds to Claudi

**Workflow:**
1. Plane task created: "Research X"
2. n8n gathers research → saves to `/root/Research/project-id/`
3. Hucki indexes documents → embeds in Convex
4. Hucki synthesizes → generates `context.md` via Claude Code
5. Billi reads context → commissions Claudi to build

### User (Secondary)
**Rob** (human) - Benefits from automated research synthesis without manual prompting

### Use Cases

1. **Automated Research Synthesis**
   - n8n workflow triggers Hucki after gathering research
   - Hucki returns context.md in <30 seconds
   - Billi/Claudi use context without re-reading raw files

2. **Project-Siloed Knowledge**
   - Each Plane task = separate workspace
   - Research for "Polymarket arb" doesn't contaminate "Next.js docs"
   - Clean isolation prevents context bleeding

3. **Query-Based Retrieval**
   - Ask specific questions: "What are the returns?"
   - Get relevant chunks + sources
   - No full synthesis needed for quick lookups

## Tech Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Vector DB** | Convex | Already deployed, handles vectors natively, easy API |
| **LLM** | Claude Code CLI | Free (subscription), local execution, no API costs |
| **Embeddings** | sentence-transformers (local) | Free, 384-dim, good enough for RAG |
| **API Framework** | Express.js | Lightweight, familiar, easy n8n integration |
| **Auth** | Bearer token | Simple, stateless, n8n-friendly |

**Key constraint:** <1000 lines of code (excluding tests)

## Requirements

### Validated
(None yet — ship to validate)

### Active

#### Workspace Management
- [ ] **WS-01**: Create workspace with unique ID
- [ ] **WS-02**: List all workspaces
- [ ] **WS-03**: Delete workspace and all vectors
- [ ] **WS-04**: Get workspace stats (doc_count, vector_count)

#### Document Indexing
- [ ] **IDX-01**: Index single file (.md, .txt, .pdf)
- [ ] **IDX-02**: Index entire directory
- [ ] **IDX-03**: Chunk documents (max 1000 chars, preserve sentences)
- [ ] **IDX-04**: Generate embeddings (sentence-transformers)
- [ ] **IDX-05**: Store vectors in Convex with workspace isolation
- [ ] **IDX-06**: Return indexing status (chunks, vectors created)

#### Context Synthesis
- [ ] **SYN-01**: Retrieve top 10 relevant chunks from Convex
- [ ] **SYN-02**: Pass chunks to Claude Code CLI
- [ ] **SYN-03**: Generate structured context.md (summary, findings, next steps)
- [ ] **SYN-04**: Save to `/root/Research/{workspace_id}/context.md`
- [ ] **SYN-05**: Complete synthesis in <30 seconds

#### Query Interface
- [ ] **QRY-01**: Query workspace with natural language
- [ ] **QRY-02**: Return relevant chunks with sources
- [ ] **QRY-03**: Optional synthesis mode (chunks → Claude answer)
- [ ] **QRY-04**: Support similarity search filters

#### System Health
- [ ] **HLT-01**: Health check endpoint
- [ ] **HLT-02**: Report Convex connection status
- [ ] **HLT-03**: Report Claude Code CLI availability
- [ ] **HLT-04**: Return system metrics (workspace/vector counts)

### Out of Scope

- ❌ **Web UI** — Headless API only (n8n is the UI)
- ❌ **Multi-user auth** — Single-tenant (Billi/Rob only)
- ❌ **Real-time chat** — Batch synthesis only
- ❌ **PDF OCR** — Text-based PDFs only (n8n handles OCR upstream)
- ❌ **Web scraping** — n8n's job
- ❌ **Audio/video** — Text only
- ❌ **Custom embeddings** — sentence-transformers is fixed (can upgrade later)

**Rationale:** Keep v1 minimal. IRCA workflow doesn't need these. Add only if validated demand.

## Success Criteria

**Ship when:**
1. All 24 Active requirements pass integration tests
2. Quality eval >80% (blind test vs human baseline)
3. 3/3 real-world IRCA workflows complete successfully
4. Performance: context synthesis <30s
5. Code: <1000 LOC (excluding tests)
6. Security: passes basic auth + isolation tests

**Don't ship if:**
- Context quality <80% (defeats core value)
- Claude Code CLI unreliable
- Convex integration flaky

## Constraints

### Must Have
- **Cost:** $0 API costs (use Claude Code, not API)
- **Security:** All data in Convex or local file system (no external services)
- **Speed:** <30s synthesis (automation-friendly)
- **Simplicity:** <1000 LOC (maintainability)

### Nice to Have
- Documentation
- Example n8n workflows
- Performance benchmarks

### Out of Bounds
- Don't build a UI (this is API-only)
- Don't add multi-user features (single-tenant)
- Don't optimize for features not in Active requirements

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Convex over LanceDB | Already deployed, handles vectors, simpler than self-hosted LanceDB | Pending validation |
| Claude Code CLI over API | Zero cost, uses existing subscription | Pending validation |
| Sentence-transformers embeddings | Free, local, good quality for RAG | Pending validation |
| Express.js API | Lightweight, n8n-friendly | Pending validation |
| <1000 LOC constraint | Forces simplicity, prevents bloat | — |

## Open Questions

1. **Convex vector index performance:** How does it handle 10k+ vectors per workspace?
2. **Claude Code CLI reliability:** Does it handle concurrent calls? Rate limits?
3. **Embedding model swap:** Easy to upgrade sentence-transformers to Voyage AI later?
4. **n8n integration patterns:** Best way to trigger Hucki from n8n? (webhook vs HTTP Request node)

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---

*Last updated: 2026-03-27 after initialization*
