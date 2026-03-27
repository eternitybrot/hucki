# Hucki v1 Roadmap

**Created:** 2026-03-27
**Target:** v1.0 Launch
**Strategy:** Coarse granularity (5 phases, vertical slices)

---

## Roadmap Overview

**5 phases** | **24 requirements** | All v1 requirements covered ✓

This roadmap builds Hucki incrementally—each phase delivers a working vertical slice that can be tested end-to-end.

---

## Phases

| # | Phase | Goal | Requirements | Success Criteria |
|---|-------|------|--------------|------------------|
| 1 | Foundation | Core API + workspace management working | WS-01 to WS-04, HLT-01 to HLT-04 | Can create/list/delete workspaces via API |
| 2 | Embedding Pipeline | Document indexing + vector storage working | IDX-01 to IDX-06 | Can index docs and store in Convex |
| 3 | Retrieval Engine | Query interface + vector search working | QRY-01, QRY-02, QRY-04 | Can query workspace and get relevant chunks |
| 4 | Synthesis Engine | Claude Code integration + context generation | SYN-01 to SYN-05, QRY-03 | Can generate context.md via Claude |
| 5 | Production Ready | Testing, eval, docs, deployment | All NFRs | Passes all evals, ready for n8n integration |

---

## Phase 1: Foundation

**Goal:** Establish core API infrastructure and workspace management

**Requirements:**
- WS-01: Create workspace
- WS-02: List workspaces
- WS-03: Delete workspace
- WS-04: Get workspace stats
- HLT-01: Health endpoint
- HLT-02: Convex connection check
- HLT-03: Claude Code availability check
- HLT-04: System metrics

**Success Criteria:**
1. API server starts and responds to /health
2. Can create workspace via POST /workspace/create
3. Workspace appears in GET /workspaces list
4. Can delete workspace via DELETE /workspace/:id
5. Convex connection verified in health check
6. All workspace operations return proper status codes

**Deliverables:**
- `src/server.js` - Express server
- `src/api/workspaces.js` - Workspace endpoints
- `src/api/health.js` - Health check endpoint
- `src/lib/convex.js` - Convex client wrapper
- `convex/schema.ts` - Convex schema with workspaces table
- `.env.example` - Environment configuration template
- Unit tests for workspace CRUD

**Technical Notes:**
- Start simple: in-memory workspace tracking, then add Convex
- Health endpoint must actually test Convex, not just return OK
- Use Convex mutations for workspace creation/deletion
- Implement bearer token auth from day 1

**UI hint**: no

---

## Phase 2: Embedding Pipeline

**Goal:** Process documents and store vector embeddings in Convex

**Requirements:**
- IDX-01: Index single file
- IDX-02: Index directory
- IDX-03: Chunk documents
- IDX-04: Generate embeddings
- IDX-05: Store vectors in Convex
- IDX-06: Return indexing status

**Success Criteria:**
1. Can upload a .md file and get chunks back
2. Chunks preserve sentence boundaries (no mid-sentence splits)
3. Embeddings are 384-dimensional vectors
4. Vectors stored in Convex with workspace_id filter
5. Can index entire directory with mixed file types
6. Indexing errors reported (don't crash on bad files)

**Deliverables:**
- `src/api/indexing.js` - Indexing endpoints
- `src/lib/embedder.js` - Python subprocess wrapper
- `src/lib/chunker.js` - Document chunking logic
- `src/lib/document-processor.js` - File reading (md, txt, pdf)
- `python/embed.py` - sentence-transformers embedding script
- `convex/schema.ts` - Add vectors table with vector index
- `convex/vectors.ts` - Vector CRUD operations
- Integration tests for full indexing pipeline

**Technical Notes:**
- Use Python subprocess for embeddings (sentence-transformers)
- PDF support: pypdf for text extraction (text-based PDFs only)
- Chunking: max 1000 chars, break on sentence boundaries
- Batch inserts to Convex (don't insert one-by-one)
- Return detailed status: {chunks_indexed, vectors_created, errors[]}

**UI hint**: no

---

## Phase 3: Retrieval Engine

**Goal:** Query workspaces and retrieve relevant chunks via vector search

**Requirements:**
- QRY-01: Query with natural language
- QRY-02: Return chunks with sources
- QRY-04: Similarity threshold filtering

**Success Criteria:**
1. Can query workspace with natural language question
2. Returns top 10 most relevant chunks
3. Results include source file paths
4. Similarity scores included in response
5. Threshold parameter filters low-relevance results
6. Query returns empty array if no matches (not error)

**Deliverables:**
- `src/api/query.js` - Query endpoints
- `src/lib/retriever.js` - Vector search wrapper
- `convex/vectors.ts` - Add vector search query
- Integration tests for query accuracy

**Technical Notes:**
- Embed the query using same embedder as documents
- Use Convex vectorSearch with workspace_id filter
- Return format: {chunks: [{text, source, score}]}
- Default threshold: 0.25 (configurable via param)
- No LLM involvement yet (just retrieval)

**UI hint**: no

---

## Phase 4: Synthesis Engine

**Goal:** Generate structured context documents using Claude Code CLI

**Requirements:**
- SYN-01: Retrieve relevant chunks
- SYN-02: Pass to Claude Code CLI
- SYN-03: Generate structured context.md
- SYN-04: Save to file system
- SYN-05: Complete in <30 seconds
- QRY-03: Synthesize mode for queries

**Success Criteria:**
1. Can trigger synthesis via POST /workspace/:id/synthesize
2. Claude Code CLI called successfully with structured prompt
3. context.md file created with all required sections
4. Synthesis completes in <30 seconds
5. Query endpoint supports mode="synthesize" (not just retrieve)
6. Handles Claude Code unavailable gracefully (error, not crash)

**Deliverables:**
- `src/api/synthesis.js` - Synthesis endpoint
- `src/lib/claude.js` - Claude Code CLI wrapper
- `src/lib/prompt-builder.js` - Build synthesis prompts
- Integration tests for synthesis quality
- Update query.js to support synthesis mode

**Technical Notes:**
- Claude Code call: `echo "[prompt]" | claude --print --no-stream`
- Prompt template: Include chunks, ask for structured output
- Sections: Executive Summary, Key Findings, Analysis, Next Steps, Sources
- File path: `/root/Research/{workspace_id}/context.md`
- Create directory if it doesn't exist
- Timeout after 60 seconds (fail gracefully)
- Query synthesis mode: embed question → search → pass to Claude

**UI hint**: no

---

## Phase 5: Production Ready

**Goal:** Testing, evaluation, documentation, and deployment readiness

**Requirements:**
- All NFRs (performance, reliability, security, maintainability)

**Success Criteria:**
1. 100% unit test coverage on core modules
2. 100% integration test pass rate (all 10 scenarios)
3. Quality eval >80% (5 blind test scenarios)
4. 3/3 real-world IRCA workflows complete successfully
5. Performance benchmarks met (<30s synthesis, <500ms search)
6. Security tests pass (auth, isolation, input validation)
7. Documentation complete (README, API docs, n8n examples)
8. Code <1000 LOC (excluding tests)
9. Linter 0 errors

**Deliverables:**
- `tests/unit/` - Complete unit test suite
- `tests/integration/` - Integration test suite
- `tests/eval/` - Quality evaluation suite
- `README.md` - Setup and usage guide
- `API.md` - Complete API documentation
- `examples/n8n/` - Example workflows
- Performance benchmarks
- Security audit report
- Deployment guide (PM2, systemd)

**Technical Notes:**
- Use EVAL.md spec for quality evaluation
- Create eval dataset (5 scenarios with ground truth)
- Blind eval: run without seeing expected outputs
- Real-world tests: Polymarket research, docs synthesis, multi-source
- Performance: measure with real workloads (not toy data)
- Document all environment variables
- Provide Docker option for easier deployment

**UI hint**: no

---

## Requirement Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| **Workspace Management** |
| WS-01 | Phase 1 | Pending |
| WS-02 | Phase 1 | Pending |
| WS-03 | Phase 1 | Pending |
| WS-04 | Phase 1 | Pending |
| **Document Indexing** |
| IDX-01 | Phase 2 | Pending |
| IDX-02 | Phase 2 | Pending |
| IDX-03 | Phase 2 | Pending |
| IDX-04 | Phase 2 | Pending |
| IDX-05 | Phase 2 | Pending |
| IDX-06 | Phase 2 | Pending |
| **Context Synthesis** |
| SYN-01 | Phase 4 | Pending |
| SYN-02 | Phase 4 | Pending |
| SYN-03 | Phase 4 | Pending |
| SYN-04 | Phase 4 | Pending |
| SYN-05 | Phase 4 | Pending |
| **Query Interface** |
| QRY-01 | Phase 3 | Pending |
| QRY-02 | Phase 3 | Pending |
| QRY-03 | Phase 4 | Pending |
| QRY-04 | Phase 3 | Pending |
| **System Health** |
| HLT-01 | Phase 1 | Pending |
| HLT-02 | Phase 1 | Pending |
| HLT-03 | Phase 1 | Pending |
| HLT-04 | Phase 1 | Pending |
| **Non-Functional** |
| All NFRs | Phase 5 | Pending |

**Coverage:** 24/24 v1 requirements mapped ✓

---

## Dependencies

### Phase Dependencies
- Phase 2 depends on Phase 1 (needs workspace management)
- Phase 3 depends on Phase 2 (needs embeddings to search)
- Phase 4 depends on Phase 3 (needs retrieval for synthesis)
- Phase 5 depends on Phases 1-4 (tests everything)

### External Dependencies
- Convex project deployed (`optimistic-pig-632` or new)
- Claude Code CLI installed and accessible
- Python 3.13+ with sentence-transformers
- Node.js 18+

---

## Risk Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Claude Code CLI unreliable | High | Fallback to mock mode for testing, document workarounds |
| Convex vector search performance | Medium | Benchmark early (Phase 3), optimize queries, consider pagination |
| Embedding quality insufficient | Medium | Run quality eval early (Phase 5), swap models if needed |
| <1000 LOC constraint too tight | Low | Ruthlessly cut features, extract to utils/ if needed |

---

## Success Metrics

### Velocity
- Target: 1 phase per 2-3 days
- Total: 10-15 days to v1.0

### Quality
- Unit tests: 100% pass rate
- Integration tests: 100% pass rate
- Quality eval: >80% score
- Real-world tests: 3/3 pass

### Performance
- Synthesis: <30s (measured)
- Search: <500ms (measured)
- API: <100ms (non-synthesis endpoints)

---

*Roadmap evolves through phase completion. See PROJECT.md Evolution section for update protocol.*
