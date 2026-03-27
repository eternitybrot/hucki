# Hucki v1 Requirements

**Last Updated:** 2026-03-27
**Milestone:** v1.0 Launch

---

## v1 Requirements

### Workspace Management (WS)

- [ ] **WS-01**: Create workspace with unique slugified ID
  - API: `POST /workspace/create {name}`
  - Returns: `{id, name, created_at}`

- [ ] **WS-02**: List all workspaces
  - API: `GET /workspaces`
  - Returns: `[{id, name, doc_count, vector_count}]`

- [ ] **WS-03**: Delete workspace and all associated data
  - API: `DELETE /workspace/:id`
  - Removes: workspace metadata + all vectors from Convex

- [ ] **WS-04**: Get workspace statistics
  - API: `GET /workspace/:id/stats`
  - Returns: `{doc_count, vector_count, last_indexed}`

### Document Indexing (IDX)

- [ ] **IDX-01**: Index single file (.md, .txt, .pdf)
  - API: `POST /workspace/:id/index {file_path}`
  - Supports: Markdown, plain text, text-based PDFs

- [ ] **IDX-02**: Index entire directory
  - API: `POST /workspace/:id/index {directory}`
  - Recursively processes all supported files

- [ ] **IDX-03**: Chunk documents intelligently
  - Max 1000 chars per chunk
  - Preserve sentence boundaries
  - No mid-word breaks

- [ ] **IDX-04**: Generate embeddings locally
  - Model: sentence-transformers (all-MiniLM-L6-v2)
  - Dimensions: 384
  - Local execution (no API calls)

- [ ] **IDX-05**: Store vectors in Convex with workspace isolation
  - Schema: workspace_id, chunk_text, embedding, source_file, chunk_index
  - Vector index: by_embedding with workspace_id filter

- [ ] **IDX-06**: Return detailed indexing status
  - Returns: `{chunks_indexed, vectors_created, errors[]}`
  - Handle partial failures gracefully

### Context Synthesis (SYN)

- [ ] **SYN-01**: Retrieve top 10 relevant chunks via vector search
  - Convex vector similarity search
  - Filter by workspace_id
  - Return chunks + sources

- [ ] **SYN-02**: Pass chunks to Claude Code CLI
  - Build structured prompt with chunks
  - Call: `echo "[prompt]" | claude --print --no-stream`
  - Handle CLI unavailable gracefully

- [ ] **SYN-03**: Generate structured context.md
  - Sections: Executive Summary, Key Findings, Detailed Analysis, Next Steps, Sources
  - Format: Clean markdown
  - Quality: actionable insights, not just summaries

- [ ] **SYN-04**: Save context to file system
  - Path: `/root/Research/{workspace_id}/context.md`
  - Create directory if needed
  - Atomic write (no partial files)

- [ ] **SYN-05**: Complete synthesis in <30 seconds
  - Measured from API call to file written
  - Timeout if Claude Code hangs

### Query Interface (QRY)

- [ ] **QRY-01**: Query workspace with natural language
  - API: `POST /workspace/:id/query {question}`
  - Embed question → vector search → return results

- [ ] **QRY-02**: Return relevant chunks with sources
  - Returns: `{chunks: [{text, source, score}]}`
  - Sorted by relevance
  - Include source file paths

- [ ] **QRY-03**: Optional synthesis mode
  - Mode: "retrieve" (chunks only) or "synthesize" (Claude answer)
  - Synthesize: chunks → Claude → answer

- [ ] **QRY-04**: Support similarity threshold filtering
  - Parameter: `threshold` (0-1, default 0.25)
  - Only return chunks above threshold

### System Health (HLT)

- [ ] **HLT-01**: Health check endpoint
  - API: `GET /health`
  - Returns: overall status (healthy/degraded/down)

- [ ] **HLT-02**: Report Convex connection status
  - Test: actual query to Convex
  - Status: connected/disconnected/error

- [ ] **HLT-03**: Report Claude Code CLI availability
  - Test: `which claude` or version check
  - Status: available/unavailable

- [ ] **HLT-04**: Return system metrics
  - Total workspaces
  - Total vectors across all workspaces
  - Uptime

---

## v2 Requirements (Deferred)

### Advanced Features
- **ADV-01**: Reranking layer for better retrieval
- **ADV-02**: Support for custom embedding models (Voyage AI)
- **ADV-03**: Streaming synthesis responses
- **ADV-04**: Batch operations (index multiple workspaces)
- **ADV-05**: Webhook callbacks on completion
- **ADV-06**: Workspace archiving/backup

### Performance Optimizations
- **PERF-01**: Embedding caching (skip re-embed unchanged files)
- **PERF-02**: Parallel chunk processing
- **PERF-03**: Vector search result caching
- **PERF-04**: Database connection pooling

### Integration Enhancements
- **INT-01**: n8n credential node for easier auth
- **INT-02**: Example workflows repository
- **INT-03**: Zapier integration
- **INT-04**: Slack bot interface

---

## Out of Scope

### Explicitly Excluded
- ❌ **UI-01**: Web interface — API-only by design
- ❌ **AUTH-01**: Multi-user authentication — Single-tenant
- ❌ **CHAT-01**: Real-time chat interface — Batch synthesis only
- ❌ **OCR-01**: PDF OCR — Text PDFs only (n8n handles OCR)
- ❌ **WEB-01**: Web scraping — n8n's responsibility
- ❌ **MEDIA-01**: Audio/video processing — Text only
- ❌ **MODEL-01**: Multiple embedding models — Fixed to sentence-transformers
- ❌ **LANG-01**: Multi-language support — English-first

**Rationale:** Keep v1 focused on core value. These features don't serve the IRCA automation use case. Add only if validated user demand.

---

## Traceability Matrix

*Updated by roadmapper*

| Requirement | Phase | Status |
|-------------|-------|--------|
| *(To be filled by roadmap creation)* |

---

## Non-Functional Requirements

### Performance
- **NFR-PERF-01**: Document indexing <2s per document (1000 words)
- **NFR-PERF-02**: Vector search <500ms for top 10 results
- **NFR-PERF-03**: Full synthesis <30s (index → search → Claude → save)
- **NFR-PERF-04**: API response time <100ms (non-synthesis endpoints)

### Scalability
- **NFR-SCALE-01**: Support 100 concurrent workspaces
- **NFR-SCALE-02**: Handle 10,000 vectors per workspace
- **NFR-SCALE-03**: Total capacity: 100,000 vectors
- **NFR-SCALE-04**: API throughput: 60 requests/min

### Reliability
- **NFR-REL-01**: 99% uptime (self-hosted)
- **NFR-REL-02**: Graceful degradation if Convex unavailable
- **NFR-REL-03**: Transaction safety for indexing (no partial state)
- **NFR-REL-04**: Automatic retry with exponential backoff

### Security
- **NFR-SEC-01**: Bearer token authentication on all endpoints
- **NFR-SEC-02**: Workspace isolation (no cross-workspace data leakage)
- **NFR-SEC-03**: Input validation (prevent injection attacks)
- **NFR-SEC-04**: Rate limiting (prevent abuse)

### Maintainability
- **NFR-MAINT-01**: <1000 lines of code (excluding tests)
- **NFR-MAINT-02**: Single Node.js process (no microservices)
- **NFR-MAINT-03**: Minimal dependencies (Convex, sentence-transformers only)
- **NFR-MAINT-04**: Comprehensive logging (winston)
- **NFR-MAINT-05**: 80% code coverage (unit + integration tests)

---

## Quality Gates

### Unit Test Pass Rate
- 100% of unit tests must pass
- No test skips or TODOs in final merge

### Integration Test Pass Rate
- 100% of integration tests must pass
- End-to-end scenarios covered

### Quality Evaluation Score
- >80% vs human baseline (blind eval)
- 5/5 test scenarios must score >32/40

### Real-World Test Pass Rate
- 3/3 actual IRCA workflows complete successfully
- No manual intervention required

### Code Quality
- Linter: 0 errors
- Complexity: cyclomatic complexity <10 per function
- Size: <1000 LOC (excluding tests, node_modules)

---

*Requirements evolve through phase transitions. See PROJECT.md Evolution section for update protocol.*
