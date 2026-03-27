# Hucki - Development Roadmap

**Milestone:** v1.0 Comprehensive
**Version:** 2.0 (Feature Parity with AnythingLLM)
**Status:** Planning Complete
**Created:** 2026-03-27

---

## Phases

- [ ] **Phase 6: Foundation** - API framework, workspace management, health monitoring
- [ ] **Phase 7: Text Document Processing** - PDF, DOCX, PPTX, XLSX, Markdown, JSON, CSV
- [ ] **Phase 8: Code File Processing** - 50+ languages with syntax-aware parsing
- [ ] **Phase 9: Media Processing** - OCR, audio transcription, video extraction
- [ ] **Phase 10: Web Connector** - Puppeteer scraping with SPA support
- [ ] **Phase 11: Data Connectors** - YouTube, GitHub, Confluence integration
- [ ] **Phase 12: Chunking & Embeddings** - Intelligent text processing and local embeddings
- [ ] **Phase 13: Vector Database** - Convex integration with workspace isolation
- [ ] **Phase 14: Synthesis Engine** - Claude Code CLI integration with 4 synthesis modes
- [ ] **Phase 15: Production Readiness** - Tests, evals, documentation, deployment

---

## Phase Details

### Phase 6: Foundation
**Goal**: API infrastructure ready to accept documents and manage workspaces
**Depends on**: Nothing (first phase)
**Requirements**: API-01, API-02, API-03, API-04, API-05, API-09, WS-01, WS-02, WS-03, WS-04, HLT-01, HLT-02, HLT-03, HLT-04, HLT-05
**Success Criteria** (what must be TRUE):
  1. User can create workspace via POST /workspace/create and receive unique slug
  2. User can list all workspaces via GET /workspaces with metadata
  3. User can delete workspace via DELETE /workspace/:slug and all associated data removed
  4. Health endpoint returns system status (Convex connected, Claude Code available, Puppeteer ready)
  5. API responds to requests with proper error handling and structured JSON responses
**Plans**: TBD
**UI hint**: yes

### Phase 7: Text Document Processing
**Goal**: System processes 8 core text formats with metadata extraction
**Depends on**: Phase 6
**Requirements**: DOC-TXT-01, DOC-TXT-02, DOC-TXT-03, DOC-TXT-04, DOC-TXT-05, DOC-TXT-06, DOC-TXT-07, DOC-TXT-08, DOC-WEB-01, DOC-WEB-02, DOC-WEB-03, DOC-ARCH-01, DOC-ARCH-02, DOC-ARCH-03
**Success Criteria** (what must be TRUE):
  1. User can upload PDF file and receive extracted text in <2s for 10-page document
  2. User can upload DOCX/PPTX/XLSX files and receive structured content (paragraphs, slides, sheets)
  3. User can upload Markdown/JSON/CSV files and receive parsed content with preserved structure
  4. User can upload HTML/XML files and receive clean text with extracted metadata
  5. User can upload ZIP/TAR archives and system recursively processes all contained files (up to 3 levels deep)
**Plans**: TBD
**UI hint**: yes

### Phase 8: Code File Processing
**Goal**: System parses 50+ programming languages with syntax awareness
**Depends on**: Phase 7
**Requirements**: DOC-CODE-01, DOC-CODE-02, DOC-CODE-03, DOC-CODE-04, DOC-CODE-05, DOC-CODE-06, DOC-CODE-07, DOC-CODE-08, DOC-CODE-09
**Success Criteria** (what must be TRUE):
  1. User can upload JavaScript/TypeScript/Python/Java/C++/Go/Rust files and receive syntax-aware parsing
  2. User can see extracted docstrings, comments, and function signatures from code files
  3. User can upload code in any of 50+ supported languages and receive proper tokenization
  4. Code structure is preserved (indentation, formatting, AST elements) in extracted text
  5. System processes 1000-line code file in <500ms with detected imports and dependencies
**Plans**: TBD

### Phase 9: Media Processing
**Goal**: System transcribes audio, performs OCR on images, extracts video content
**Depends on**: Phase 7
**Requirements**: DOC-MEDIA-01, DOC-MEDIA-02, DOC-MEDIA-03, DOC-MEDIA-04, DOC-MEDIA-05
**Success Criteria** (what must be TRUE):
  1. User can upload PNG/JPG/TIFF images with text and receive OCR-extracted content with >90% accuracy
  2. User can upload MP3/WAV/M4A audio files and receive local Whisper transcription with <10% WER
  3. User can upload audio with automatic fallback to cloud Whisper API if local fails
  4. User can upload MP4/AVI/MOV video files and system extracts audio for transcription
  5. System processes 1-minute audio in <10 seconds locally or video frames for OCR
**Plans**: TBD

### Phase 10: Web Connector
**Goal**: System scrapes websites with JavaScript rendering and robots.txt compliance
**Depends on**: Phase 6
**Requirements**: CONN-WEB-01, CONN-WEB-02, CONN-WEB-03, CONN-WEB-04, CONN-WEB-05, CONN-WEB-06, CONN-WEB-07, CONN-WEB-08, CONN-WEB-09, CONN-WEB-10, CONN-GEN-01, CONN-GEN-02, CONN-GEN-03
**Success Criteria** (what must be TRUE):
  1. User can provide URL and receive scraped content from static and SPA websites (React, Vue, Angular)
  2. User can see JavaScript-rendered content with configurable wait strategies (networkidle0, domcontentloaded)
  3. System respects robots.txt and blocks local IP scraping by default (ALLOW_LOCAL_IPS=false)
  4. User can crawl sitemaps and perform recursive crawling with depth limits and same-domain enforcement
  5. System applies rate limiting (default 1 req/sec), custom headers, and proxy support
**Plans**: TBD

### Phase 11: Data Connectors
**Goal**: System extracts content from YouTube, GitHub, and Confluence
**Depends on**: Phase 10
**Requirements**: CONN-YT-01, CONN-YT-02, CONN-YT-03, CONN-YT-04, CONN-GH-01, CONN-GH-02, CONN-GH-03, CONN-GH-04, CONN-GH-05, CONN-CONF-01, CONN-CONF-02, CONN-CONF-03, CONN-CONF-04
**Success Criteria** (what must be TRUE):
  1. User can provide YouTube URL and receive transcript with timestamps and multi-language support
  2. User can provide GitHub repo URL and system clones, processes README/docs/code files (public or OAuth for private)
  3. User can select specific branches and extract issues/PRs via GitHub API
  4. User can provide Confluence page URL and receive scraped wiki content with preserved hierarchy
  5. System falls back to Whisper transcription if YouTube transcript unavailable
**Plans**: TBD

### Phase 12: Chunking & Embeddings
**Goal**: Text is intelligently chunked and embedded locally with zero API cost
**Depends on**: Phases 7, 8, 9
**Requirements**: CHUNK-01, CHUNK-02, CHUNK-03, CHUNK-04, CHUNK-05, CHUNK-06, CHUNK-07, CHUNK-08, CHUNK-09, CHUNK-10, CHUNK-11, EMB-01, EMB-02, EMB-03, EMB-04, EMB-05, EMB-06, EMB-07, EMB-08
**Success Criteria** (what must be TRUE):
  1. User can configure chunk size (100-5000 tokens) and overlap (0-500 tokens) per workspace
  2. Text is chunked at sentence boundaries without mid-sentence splits, with paragraph awareness
  3. Code blocks and markdown structure are preserved during chunking (headings, lists, links)
  4. System generates 384-dim or 1024-dim embeddings locally using all-MiniLM-L6-v2 or alternative models
  5. System processes 100+ documents in batch with embedding caching (>80% cache hit rate) and L2 normalization
**Plans**: TBD

### Phase 13: Vector Database
**Goal**: Vectors stored in Convex with workspace isolation and fast search
**Depends on**: Phase 12
**Requirements**: VDB-01, VDB-02, VDB-03, VDB-04, VDB-05, VDB-06, VDB-07, VDB-08, NFR-SCALE-01, NFR-SCALE-02, NFR-SCALE-03, NFR-SCALE-04
**Success Criteria** (what must be TRUE):
  1. User can insert 100+ vectors in batch to workspace in <500ms
  2. User can search vectors with cosine similarity and receive top-k results in <100ms (p95)
  3. Workspace isolation enforced (no cross-workspace data leaks, filter by workspace_id)
  4. User can delete workspace and all associated vectors are removed atomically
  5. System handles 100k+ vectors per workspace without performance degradation
**Plans**: TBD

### Phase 14: Synthesis Engine
**Goal**: Claude Code CLI generates high-quality synthesis with zero API cost
**Depends on**: Phase 13
**Requirements**: SYN-01, SYN-02, SYN-03, SYN-04, SYN-05, SYN-06, SYN-07, SYN-08, SYN-09, SYN-10, API-06, API-07, API-08, NFR-PERF-03
**Success Criteria** (what must be TRUE):
  1. User can upload document via POST /workspace/:slug/upload and receive processing status
  2. User can scrape URL via POST /workspace/:slug/scrape with connector type selection
  3. User can query workspace via POST /workspace/:slug/query and receive synthesis
  4. User can select synthesis mode (summarize, FAQ, insights, briefing, custom)
  5. Synthesis completes in <30s for 10k tokens context (p95) with markdown output and source citations
**Plans**: TBD

### Phase 15: Production Readiness
**Goal**: System passes all tests, meets performance targets, has comprehensive documentation
**Depends on**: Phases 6 through 14
**Requirements**: NFR-PERF-01, NFR-PERF-02, NFR-PERF-04, NFR-PERF-05, NFR-SEC-01, NFR-SEC-02, NFR-SEC-03, NFR-SEC-04, NFR-REL-01, NFR-REL-02, NFR-REL-03, NFR-REL-04, NFR-MAINT-01, NFR-MAINT-02, NFR-MAINT-03, NFR-MAINT-04
**Success Criteria** (what must be TRUE):
  1. All unit tests pass (100% coverage for file processors, connectors, chunking, embeddings)
  2. All integration tests pass (end-to-end upload → vector → synthesis pipelines work)
  3. Quality eval score >80% (blind test across 5 real-world scenarios from EVAL.md)
  4. Performance targets met (PDF <2s, search <100ms, synthesis <30s, 10 concurrent uploads)
  5. Code size <5000 LOC, test coverage >80%, comprehensive README and API docs, deployment guide ready
**Plans**: TBD
**UI hint**: yes

---

## Progress Table

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 6. Foundation | 0/? | Not started | - |
| 7. Text Document Processing | 0/? | Not started | - |
| 8. Code File Processing | 0/? | Not started | - |
| 9. Media Processing | 0/? | Not started | - |
| 10. Web Connector | 0/? | Not started | - |
| 11. Data Connectors | 0/? | Not started | - |
| 12. Chunking & Embeddings | 0/? | Not started | - |
| 13. Vector Database | 0/? | Not started | - |
| 14. Synthesis Engine | 0/? | Not started | - |
| 15. Production Readiness | 0/? | Not started | - |

---

## Notes

**Granularity:** Coarse (from config.json)
**Phase numbering:** Continues from previous milestone (last phase was 5, starting at 6)
**Total requirements:** 120+ mapped across 10 phases

**Key architectural flows:**
1. **Document Upload:** Phase 7/8/9 (processors) → Phase 12 (chunking/embeddings) → Phase 13 (vector storage)
2. **Web Scraping:** Phase 10/11 (connectors) → Phase 12 (chunking/embeddings) → Phase 13 (vector storage)
3. **Synthesis:** Phase 13 (vectors) → Phase 14 (Claude Code CLI) → structured output
4. **Foundation:** Phase 6 (API + workspaces) supports all other phases

**Critical path:**
- Phase 6 (Foundation) unlocks all document processing and connector phases
- Phases 7, 8, 9 (processors) and Phases 10, 11 (connectors) run in parallel once Phase 6 complete
- Phase 12 (chunking/embeddings) required for Phase 13 (vector DB)
- Phase 13 (vectors) required for Phase 14 (synthesis)
- Phase 15 (production) validates entire system

**Parallelization opportunities:**
- Phases 7, 8, 9 can be developed concurrently (different processors)
- Phases 10, 11 can be developed concurrently (different connectors)
- Phase 12 can start as soon as any processor (7, 8, or 9) completes

**UI phases detected:**
- Phase 6: Dashboard interface for workspace management, health monitoring
- Phase 7: Document upload interface with format selection
- Phase 15: Production dashboard with metrics, logs, performance monitoring

---

*Roadmap created: 2026-03-27*
*Next step: `/gsd:plan-phase 6`*
