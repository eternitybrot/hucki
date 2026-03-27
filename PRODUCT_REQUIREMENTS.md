# Hucki - Product Requirements Document

**Product Name:** Hucki (IRCA RAG Engine)
**Version:** 1.0.0
**Date:** 2026-03-27
**Owner:** Claudi (Billi's CTO)

---

## Vision Statement

Hucki is a lightweight, project-siloed RAG engine that uses Claude Code and Convex to synthesize research into actionable context—enabling the IRCA (Ideate → Research → Context → Action) workflow with zero API costs and maximum security.

---

## Core Problem

**Current pain:**
- NotebookLM requires Google credentials (security risk)
- AnythingLLM uses Claude API (costs money)
- No existing solution uses Claude Code directly (free, already subscribed)
- No project-siloed RAG that integrates with n8n automation

**What Hucki solves:**
✅ Uses Claude Code CLI (your existing subscription)
✅ Stores vectors in Convex (already deployed)
✅ Project-siloed workspaces (research isolation)
✅ Headless API for n8n integration
✅ Zero external API costs
✅ Maximum data privacy

---

## Product Principles

1. **Simplicity First:** Minimal code, maximum value
2. **Claude Code Native:** Built around Claude Code CLI, not APIs
3. **Project-Siloed:** Every research project is isolated
4. **Automation-Ready:** Designed for n8n, not humans
5. **Zero Lock-In:** Open source, self-hosted, portable data

---

## User Stories

### Primary User: Billi (AI Agent)

**As Billi, I want to:**
1. Create a research workspace for a new idea
2. Upload research files to that workspace
3. Query the workspace for synthesized context
4. Get a `context.md` file I can hand to Claudi for execution
5. Isolate research so different projects don't contaminate each other

### Secondary User: Rob (via n8n)

**As Rob, I want to:**
1. Drop research files in a folder
2. Have the system automatically index them
3. Get a summary without manual prompting
4. Trust that my research stays private and local

---

## Functional Requirements

### FR-1: Workspace Management

**Description:** Create isolated research workspaces (projects)

**Acceptance Criteria:**
- [ ] Can create workspace with unique ID
- [ ] Can list all workspaces
- [ ] Can delete workspace and all associated data
- [ ] Each workspace has metadata: name, created_at, doc_count, vector_count
- [ ] Workspace IDs are slugified (e.g., `plane-123` or `polymarket-research`)

**API:**
```
POST   /workspace/create         → {id, name}
GET    /workspaces               → [{id, name, doc_count}]
DELETE /workspace/:id            → {success}
GET    /workspace/:id/stats      → {doc_count, vector_count, last_indexed}
```

---

### FR-2: Document Indexing

**Description:** Process research files and store embeddings in Convex

**Acceptance Criteria:**
- [ ] Accepts file path or directory
- [ ] Supports: `.md`, `.txt`, `.pdf` (MVP)
- [ ] Chunks documents (max 1000 chars per chunk)
- [ ] Generates embeddings (local: sentence-transformers)
- [ ] Stores in Convex with workspace isolation
- [ ] Returns index status (success, chunk_count, vector_count)

**API:**
```
POST   /workspace/:id/index      → {chunks_indexed, vectors_created}
  Body: {file_path: "/root/Research/plane-123/notes.md"}
  OR:   {directory: "/root/Research/plane-123/"}
```

**Convex Schema:**
```typescript
// convex/schema.ts
export default defineSchema({
  vectors: defineTable({
    workspace_id: v.string(),
    chunk_text: v.string(),
    embedding: v.array(v.float64()),  // 384-dim (sentence-transformers)
    source_file: v.string(),
    chunk_index: v.number(),
    created_at: v.number(),
  })
    .index("by_workspace", ["workspace_id"])
    .vectorIndex("by_embedding", {
      vectorField: "embedding",
      dimensions: 384,
      filterFields: ["workspace_id"]
    }),

  workspaces: defineTable({
    id: v.string(),               // slugified ID
    name: v.string(),
    doc_count: v.number(),
    vector_count: v.number(),
    created_at: v.number(),
    last_indexed: v.number(),
  })
    .index("by_id", ["id"]),
});
```

---

### FR-3: Context Synthesis (Claude Code)

**Description:** Query workspace and generate `context.md` using Claude Code CLI

**Acceptance Criteria:**
- [ ] Accepts query or uses default: "Summarize all research"
- [ ] Retrieves top 10 relevant chunks from Convex vector search
- [ ] Passes chunks to Claude Code CLI for synthesis
- [ ] Claude Code generates structured context.md
- [ ] Saves to `/root/Research/:workspace_id/context.md`
- [ ] Returns file path

**API:**
```
POST   /workspace/:id/synthesize  → {context_file, chunk_count}
  Body: {query: "What are the key findings?"}  # Optional
```

**Claude Code Prompt Template:**
```markdown
You are a research synthesis specialist.

Read the following research chunks and create a comprehensive context document.

RESEARCH CHUNKS:
{chunks}

Generate a context document with:
1. Executive Summary (2-3 sentences)
2. Key Findings (bullet points)
3. Detailed Analysis (organized by topic)
4. Actionable Next Steps (specific, prioritized)
5. Open Questions (if any)
6. Sources Referenced (list of files)

Output to: /root/Research/{workspace_id}/context.md
```

---

### FR-4: Query Interface (RAG)

**Description:** Ask questions against workspace knowledge base

**Acceptance Criteria:**
- [ ] Accepts natural language query
- [ ] Performs vector similarity search in Convex
- [ ] Returns top N chunks with sources
- [ ] Optionally synthesizes answer via Claude Code
- [ ] Returns structured response with citations

**API:**
```
POST   /workspace/:id/query       → {answer, sources[], chunks[]}
  Body: {
    question: "What is the mean reversion strategy?",
    mode: "retrieve" | "synthesize"  # retrieve=chunks only, synthesize=Claude answer
  }
```

---

### FR-5: Health & Monitoring

**Description:** System health checks and metrics

**Acceptance Criteria:**
- [ ] Health endpoint returns system status
- [ ] Reports Convex connection status
- [ ] Reports Claude Code availability
- [ ] Returns total workspace/vector counts

**API:**
```
GET    /health                    → {status, convex, claude, stats}
```

---

## Non-Functional Requirements

### NFR-1: Performance
- Document indexing: <2s per document (1000 words)
- Vector search: <500ms for top 10 results
- Synthesis: <30s for full context generation

### NFR-2: Scalability
- Support up to 100 workspaces
- Support up to 10,000 vectors per workspace
- Support up to 1GB total research files

### NFR-3: Security
- All data in Convex (encrypted at rest)
- No external API calls (except Convex)
- Claude Code runs locally (no data leaves server)
- API requires bearer token authentication

### NFR-4: Reliability
- 99% uptime (self-hosted)
- Graceful degradation if Convex unavailable
- Transaction safety for indexing operations

### NFR-5: Maintainability
- <1000 lines of code total
- Single Node.js process
- No external dependencies except: Convex, sentence-transformers
- Comprehensive logging (winston)

---

## Technical Architecture

### Stack
```
┌─────────────────────────────────────────────────────┐
│                   Hucki Stack                       │
└─────────────────────────────────────────────────────┘

API Layer (Express.js)
    ↓
Core Services
    ├─ Workspace Manager (CRUD)
    ├─ Document Processor (chunking)
    ├─ Embedder (sentence-transformers via Python)
    ├─ Vector Store (Convex client)
    └─ Synthesizer (Claude Code CLI caller)
    ↓
Storage
    ├─ Convex (vectors + workspace metadata)
    └─ File System (/root/Research/{workspace_id}/)
```

### Data Flow
```
1. CREATE WORKSPACE
   n8n → POST /workspace/create
      → Convex.insert(workspaces)
      → Return workspace_id

2. INDEX DOCUMENTS
   n8n → POST /workspace/:id/index {directory}
      → Read files from directory
      → Chunk text (1000 chars)
      → Generate embeddings (Python subprocess)
      → Convex.insert(vectors)
      → Update workspace.doc_count

3. SYNTHESIZE CONTEXT
   n8n → POST /workspace/:id/synthesize
      → Convex.vectorSearch(workspace_id, query_embedding)
      → Get top 10 chunks
      → Build prompt with chunks
      → Call Claude Code CLI
      → Save context.md
      → Return file_path

4. QUERY
   n8n → POST /workspace/:id/query
      → Convex.vectorSearch(workspace_id, question_embedding)
      → Return chunks + sources
```

---

## Success Metrics

### Launch Criteria (v1.0)
- [ ] 5/5 core functional requirements implemented
- [ ] Passes all eval tests (see EVAL.md)
- [ ] End-to-end n8n integration working
- [ ] <1000 lines of code
- [ ] Zero external API costs
- [ ] Documentation complete

### Post-Launch KPIs
- **Speed:** Context generation <30s
- **Quality:** >80% eval score vs human baseline
- **Reliability:** 99% success rate on indexing
- **Cost:** $0/month API costs

---

## Out of Scope (v1.0)

### Explicitly NOT included:
- ❌ Web UI (headless only)
- ❌ Multi-user auth (single-tenant)
- ❌ Real-time chat interface
- ❌ PDF OCR (only text-based PDFs)
- ❌ Web scraping (n8n handles this)
- ❌ Audio/video processing
- ❌ Multi-language support
- ❌ Custom embeddings (fixed to sentence-transformers)

### Future Considerations (v2.0+):
- Multi-tenant support
- Advanced PDF processing
- Reranking layer
- Streaming responses
- Web UI for debugging

---

## Dependencies

### Required Infrastructure
- [x] Convex project deployed (`optimistic-pig-632` or new)
- [x] Claude Code CLI access
- [x] Python 3.13 + sentence-transformers
- [x] Node.js 18+
- [x] n8n running

### External Services
- Convex (vector DB + metadata)
- Claude Code (synthesis)

### Libraries
```json
{
  "dependencies": {
    "express": "^4.21.2",
    "convex": "^1.0.0",
    "winston": "^3.11.0",
    "joi": "^17.11.0",
    "dotenv": "^16.0.3"
  }
}
```

### Python Dependencies
```
sentence-transformers==2.5.1
```

---

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Claude Code CLI changes | High | Low | Document expected interface, fallback to API |
| Convex rate limits | Medium | Low | Batch operations, exponential backoff |
| Embedding quality poor | Medium | Medium | Eval-driven iteration, allow model swapping |
| n8n integration complexity | Low | Medium | Provide example workflows |

---

## Timeline

### Phase 1: Core Build (Week 1)
- Days 1-2: Setup, schema, basic API
- Days 3-4: Indexing + embeddings
- Day 5: Claude Code integration
- Days 6-7: Testing + fixes

### Phase 2: Integration (Week 2)
- Days 1-2: n8n workflows
- Days 3-4: End-to-end testing
- Day 5: Eval suite
- Days 6-7: Documentation

### Phase 3: Production (Week 3)
- Days 1-2: Error handling, logging
- Days 3-4: Performance optimization
- Day 5: Security audit
- Days 6-7: Deploy + monitoring

---

## Glossary

**Workspace:** Isolated research project (e.g., "plane-123")
**Chunk:** Text segment (max 1000 chars) from a document
**Vector:** 384-dimensional embedding of a chunk
**Embedding:** Numerical representation of text for similarity search
**Synthesis:** Claude Code generating context.md from chunks
**IRCA:** Ideate → Research → Context → Action workflow

---

## Approvals

- [ ] Product Requirements (Claudi)
- [ ] Technical Architecture (Claudi)
- [ ] Eval Spec (next document)
- [ ] Build Plan (GSD)
- [ ] Security Review (Claudi)
- [ ] Final Sign-Off (Billi/Rob)

---

**Next Document: EVAL.md** (evaluation criteria & test cases)
