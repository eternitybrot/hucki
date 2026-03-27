# Hucki v1.0 Comprehensive - Requirements

**Last Updated:** 2026-03-27
**Milestone:** v1.0 Comprehensive (Feature Parity with AnythingLLM)
**Total Requirements:** 120+

---

## Workspace Management (WS)

- [ ] **WS-01**: Create workspace with unique slugified ID
- [ ] **WS-02**: List all workspaces with metadata
- [ ] **WS-03**: Delete workspace and all vectors
- [ ] **WS-04**: Get workspace stats (doc_count, vector_count)

## Document Processing: Text (DOC-TXT)

- [ ] **DOC-TXT-01**: Process PDF files (pdf-parse)
- [ ] **DOC-TXT-02**: Process Word documents (mammoth)
- [ ] **DOC-TXT-03**: Process PowerPoint presentations (officeparser)
- [ ] **DOC-TXT-04**: Process Excel spreadsheets (officeparser)
- [ ] **DOC-TXT-05**: Process plain text files
- [ ] **DOC-TXT-06**: Process Markdown files (marked.js)
- [ ] **DOC-TXT-07**: Process JSON files (JSON.parse)
- [ ] **DOC-TXT-08**: Process CSV files (csv-parse)

## Document Processing: Code (DOC-CODE)

- [ ] **DOC-CODE-01**: Syntax-aware parsing for 50+ languages
- [ ] **DOC-CODE-02**: Preserve code structure (AST parsing)
- [ ] **DOC-CODE-03**: Extract docstrings and comments
- [ ] **DOC-CODE-04**: Detect imports and dependencies
- [ ] **DOC-CODE-05**: Support web languages (JS, TS, JSX, TSX, HTML, CSS)
- [ ] **DOC-CODE-06**: Support backend languages (Python, Java, C++, Go, Rust, etc.)
- [ ] **DOC-CODE-07**: Support data languages (SQL, R, Julia)
- [ ] **DOC-CODE-08**: Support config formats (YAML, TOML, INI, XML)
- [ ] **DOC-CODE-09**: Support shell scripts (Bash, Zsh, PowerShell)

## Document Processing: Web (DOC-WEB)

- [ ] **DOC-WEB-01**: Parse HTML files (cheerio)
- [ ] **DOC-WEB-02**: Parse XML files (xml2js)
- [ ] **DOC-WEB-03**: Extract metadata (Open Graph, Twitter Cards)

## Document Processing: Media (DOC-MEDIA)

- [ ] **DOC-MEDIA-01**: OCR images (PNG, JPG, GIF, BMP, TIFF via tesseract.js)
- [ ] **DOC-MEDIA-02**: Transcribe audio locally (MP3, WAV, M4A via Whisper)
- [ ] **DOC-MEDIA-03**: Transcribe audio via cloud (OpenAI Whisper API fallback)
- [ ] **DOC-MEDIA-04**: Extract audio from video (FFmpeg)
- [ ] **DOC-MEDIA-05**: OCR video frames (FFmpeg + tesseract)

## Document Processing: Archives (DOC-ARCH)

- [ ] **DOC-ARCH-01**: Extract ZIP archives (adm-zip)
- [ ] **DOC-ARCH-02**: Extract TAR archives (tar-stream)
- [ ] **DOC-ARCH-03**: Recursive extraction (nested archives, 3 levels)

## Data Connectors: Web Scraping (CONN-WEB)

- [ ] **CONN-WEB-01**: Scrape single URL (Puppeteer)
- [ ] **CONN-WEB-02**: Render JavaScript (SPA support)
- [ ] **CONN-WEB-03**: Wait for dynamic content (networkidle0, domcontentloaded)
- [ ] **CONN-WEB-04**: Custom headers (User-Agent, cookies)
- [ ] **CONN-WEB-05**: Sitemap crawling (parse sitemap.xml)
- [ ] **CONN-WEB-06**: Recursive crawling (depth-limited, same-domain)
- [ ] **CONN-WEB-07**: Respect robots.txt (robots-parser)
- [ ] **CONN-WEB-08**: Rate limiting (configurable delay, default 1s)
- [ ] **CONN-WEB-09**: Screenshot capture (page.screenshot)
- [ ] **CONN-WEB-10**: Proxy support (HTTP/SOCKS)

## Data Connectors: YouTube (CONN-YT)

- [ ] **CONN-YT-01**: Extract transcripts (youtubei.js)
- [ ] **CONN-YT-02**: Multi-language support (auto-detect)
- [ ] **CONN-YT-03**: Timestamp mapping (preserve video timestamps)
- [ ] **CONN-YT-04**: Fallback to Whisper (if no transcript, download audio)

## Data Connectors: GitHub (CONN-GH)

- [ ] **CONN-GH-01**: Clone repositories (git clone or GitHub API)
- [ ] **CONN-GH-02**: Select branches (default or user-specified)
- [ ] **CONN-GH-03**: Process code + docs (README, docs/, code files)
- [ ] **CONN-GH-04**: Extract issues/PRs (GitHub API)
- [ ] **CONN-GH-05**: OAuth support (GitHub token)

## Data Connectors: Confluence (CONN-CONF)

- [ ] **CONN-CONF-01**: Scrape wiki pages (Confluence REST API)
- [ ] **CONN-CONF-02**: Preserve hierarchy (parent/child pages)
- [ ] **CONN-CONF-03**: Download attachments (API attachment endpoints)
- [ ] **CONN-CONF-04**: Authentication (API token or OAuth)

## Data Connectors: Generic (CONN-GEN)

- [ ] **CONN-GEN-01**: Follow redirects (3xx handling)
- [ ] **CONN-GEN-02**: Parse RSS feeds (rss-parser)
- [ ] **CONN-GEN-03**: Extract article content (readability.js)

## Text Processing & Chunking (CHUNK)

- [ ] **CHUNK-01**: Configurable chunk size (default 1000 tokens, range 100-5000)
- [ ] **CHUNK-02**: Configurable overlap (default 20 tokens, range 0-500)
- [ ] **CHUNK-03**: Sentence boundary preservation (no mid-sentence splits)
- [ ] **CHUNK-04**: Paragraph-aware chunking
- [ ] **CHUNK-05**: Min chunk size (default 100 tokens)
- [ ] **CHUNK-06**: Whitespace normalization
- [ ] **CHUNK-07**: Language detection (franc or langdetect)
- [ ] **CHUNK-08**: Markdown preservation (headings, lists, code blocks)
- [ ] **CHUNK-09**: Code syntax preservation
- [ ] **CHUNK-10**: Link extraction (preserve for citations)
- [ ] **CHUNK-11**: Metadata extraction (title, author, date, source URL)

## Embedding Pipeline (EMB)

- [ ] **EMB-01**: Local embeddings (all-MiniLM-L6-v2, 384-dim)
- [ ] **EMB-02**: Alternative model 1 (bge-small-en-v1.5)
- [ ] **EMB-03**: Alternative model 2 (e5-large, 1024-dim)
- [ ] **EMB-04**: Cloud fallback (OpenAI text-embedding-3-small)
- [ ] **EMB-05**: Batch processing (100+ docs simultaneously)
- [ ] **EMB-06**: Caching (skip re-embedding unchanged docs)
- [ ] **EMB-07**: Metadata embedding (titles/headers separately)
- [ ] **EMB-08**: L2 normalization (cosine similarity)

## Vector Database: Convex (VDB)

- [ ] **VDB-01**: Convex schema (workspaces + vectors tables)
- [ ] **VDB-02**: Vector index (384-dim or 1024-dim)
- [ ] **VDB-03**: Workspace isolation (filter by workspace_id)
- [ ] **VDB-04**: Batch insert (100+ vectors)
- [ ] **VDB-05**: Vector search (cosine similarity, top-k)
- [ ] **VDB-06**: Delete vectors (by workspace_id or vector ID)
- [ ] **VDB-07**: Update vectors (replace existing)
- [ ] **VDB-08**: Namespace management (create/delete workspaces)

## Synthesis Engine: Claude Code (SYN)

- [ ] **SYN-01**: CLI invocation (echo "prompt" | claude --print --no-stream)
- [ ] **SYN-02**: Context injection (inject top-k retrieved chunks)
- [ ] **SYN-03**: Prompt engineering (IRCA-specific prompts)
- [ ] **SYN-04**: Output parsing (extract markdown output)
- [ ] **SYN-05**: Error handling (retry on failure, max 3 attempts)
- [ ] **SYN-06**: Summarize mode (executive summary)
- [ ] **SYN-07**: FAQ mode (Q&A pairs)
- [ ] **SYN-08**: Insights mode (bullet points)
- [ ] **SYN-09**: Briefing mode (full structured report)
- [ ] **SYN-10**: Custom mode (user-defined prompts)

## API Endpoints (API)

- [ ] **API-01**: POST /workspace/create (create workspace)
- [ ] **API-02**: GET /workspaces (list workspaces)
- [ ] **API-03**: GET /workspace/:slug (get workspace)
- [ ] **API-04**: DELETE /workspace/:slug (delete workspace)
- [ ] **API-05**: PATCH /workspace/:slug (update metadata)
- [ ] **API-06**: POST /workspace/:slug/upload (upload document)
- [ ] **API-07**: POST /workspace/:slug/scrape (scrape URL)
- [ ] **API-08**: POST /workspace/:slug/query (query and synthesize)
- [ ] **API-09**: GET /health (health check)

## System Health (HLT)

- [ ] **HLT-01**: Health check endpoint
- [ ] **HLT-02**: Convex connection status
- [ ] **HLT-03**: Claude Code CLI availability
- [ ] **HLT-04**: Puppeteer status
- [ ] **HLT-05**: System metrics (workspaces, vectors)

## Non-Functional Requirements: Performance (NFR-PERF)

- [ ] **NFR-PERF-01**: Document processing <2s for 10-page PDF
- [ ] **NFR-PERF-02**: Vector search <100ms (p95)
- [ ] **NFR-PERF-03**: Synthesis <30s for 10k tokens context
- [ ] **NFR-PERF-04**: API response <500ms (p95, non-synthesis)
- [ ] **NFR-PERF-05**: Concurrent uploads (10 simultaneous)

## Non-Functional Requirements: Scalability (NFR-SCALE)

- [ ] **NFR-SCALE-01**: Vectors per workspace (100k+)
- [ ] **NFR-SCALE-02**: Total workspaces (1000+)
- [ ] **NFR-SCALE-03**: Concurrent workers (10)
- [ ] **NFR-SCALE-04**: Max file size (100MB)

## Non-Functional Requirements: Security (NFR-SEC)

- [ ] **NFR-SEC-01**: No local IP scraping (ALLOW_LOCAL_IPS=false)
- [ ] **NFR-SEC-02**: API key security (store in Billi Vault)
- [ ] **NFR-SEC-03**: Workspace isolation (Convex RLS)
- [ ] **NFR-SEC-04**: Rate limiting (100 req/min per workspace)

## Non-Functional Requirements: Reliability (NFR-REL)

- [ ] **NFR-REL-01**: Uptime (99%)
- [ ] **NFR-REL-02**: Error handling (graceful degradation)
- [ ] **NFR-REL-03**: Retry logic (3 attempts max)
- [ ] **NFR-REL-04**: Logging (Winston, structured logs)

## Non-Functional Requirements: Maintainability (NFR-MAINT)

- [ ] **NFR-MAINT-01**: Code size (<5000 LOC)
- [ ] **NFR-MAINT-02**: Test coverage (>80%)
- [ ] **NFR-MAINT-03**: Documentation (README + API docs)
- [ ] **NFR-MAINT-04**: Dependencies (<50 npm packages)

---

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| **Workspace Management** | | |
| WS-01 | Phase 6 | Pending |
| WS-02 | Phase 6 | Pending |
| WS-03 | Phase 6 | Pending |
| WS-04 | Phase 6 | Pending |
| **Text Documents** | | |
| DOC-TXT-01 | Phase 7 | Pending |
| DOC-TXT-02 | Phase 7 | Pending |
| DOC-TXT-03 | Phase 7 | Pending |
| DOC-TXT-04 | Phase 7 | Pending |
| DOC-TXT-05 | Phase 7 | Pending |
| DOC-TXT-06 | Phase 7 | Pending |
| DOC-TXT-07 | Phase 7 | Pending |
| DOC-TXT-08 | Phase 7 | Pending |
| **Code Files** | | |
| DOC-CODE-01 | Phase 8 | Pending |
| DOC-CODE-02 | Phase 8 | Pending |
| DOC-CODE-03 | Phase 8 | Pending |
| DOC-CODE-04 | Phase 8 | Pending |
| DOC-CODE-05 | Phase 8 | Pending |
| DOC-CODE-06 | Phase 8 | Pending |
| DOC-CODE-07 | Phase 8 | Pending |
| DOC-CODE-08 | Phase 8 | Pending |
| DOC-CODE-09 | Phase 8 | Pending |
| **Web Files** | | |
| DOC-WEB-01 | Phase 7 | Pending |
| DOC-WEB-02 | Phase 7 | Pending |
| DOC-WEB-03 | Phase 7 | Pending |
| **Media Files** | | |
| DOC-MEDIA-01 | Phase 9 | Pending |
| DOC-MEDIA-02 | Phase 9 | Pending |
| DOC-MEDIA-03 | Phase 9 | Pending |
| DOC-MEDIA-04 | Phase 9 | Pending |
| DOC-MEDIA-05 | Phase 9 | Pending |
| **Archives** | | |
| DOC-ARCH-01 | Phase 7 | Pending |
| DOC-ARCH-02 | Phase 7 | Pending |
| DOC-ARCH-03 | Phase 7 | Pending |
| **Web Scraping** | | |
| CONN-WEB-01 | Phase 10 | Pending |
| CONN-WEB-02 | Phase 10 | Pending |
| CONN-WEB-03 | Phase 10 | Pending |
| CONN-WEB-04 | Phase 10 | Pending |
| CONN-WEB-05 | Phase 10 | Pending |
| CONN-WEB-06 | Phase 10 | Pending |
| CONN-WEB-07 | Phase 10 | Pending |
| CONN-WEB-08 | Phase 10 | Pending |
| CONN-WEB-09 | Phase 10 | Pending |
| CONN-WEB-10 | Phase 10 | Pending |
| **YouTube** | | |
| CONN-YT-01 | Phase 11 | Pending |
| CONN-YT-02 | Phase 11 | Pending |
| CONN-YT-03 | Phase 11 | Pending |
| CONN-YT-04 | Phase 11 | Pending |
| **GitHub** | | |
| CONN-GH-01 | Phase 11 | Pending |
| CONN-GH-02 | Phase 11 | Pending |
| CONN-GH-03 | Phase 11 | Pending |
| CONN-GH-04 | Phase 11 | Pending |
| CONN-GH-05 | Phase 11 | Pending |
| **Confluence** | | |
| CONN-CONF-01 | Phase 11 | Pending |
| CONN-CONF-02 | Phase 11 | Pending |
| CONN-CONF-03 | Phase 11 | Pending |
| CONN-CONF-04 | Phase 11 | Pending |
| **Generic Connectors** | | |
| CONN-GEN-01 | Phase 10 | Pending |
| CONN-GEN-02 | Phase 10 | Pending |
| CONN-GEN-03 | Phase 10 | Pending |
| **Chunking** | | |
| CHUNK-01 | Phase 12 | Pending |
| CHUNK-02 | Phase 12 | Pending |
| CHUNK-03 | Phase 12 | Pending |
| CHUNK-04 | Phase 12 | Pending |
| CHUNK-05 | Phase 12 | Pending |
| CHUNK-06 | Phase 12 | Pending |
| CHUNK-07 | Phase 12 | Pending |
| CHUNK-08 | Phase 12 | Pending |
| CHUNK-09 | Phase 12 | Pending |
| CHUNK-10 | Phase 12 | Pending |
| CHUNK-11 | Phase 12 | Pending |
| **Embeddings** | | |
| EMB-01 | Phase 12 | Pending |
| EMB-02 | Phase 12 | Pending |
| EMB-03 | Phase 12 | Pending |
| EMB-04 | Phase 12 | Pending |
| EMB-05 | Phase 12 | Pending |
| EMB-06 | Phase 12 | Pending |
| EMB-07 | Phase 12 | Pending |
| EMB-08 | Phase 12 | Pending |
| **Vector Database** | | |
| VDB-01 | Phase 13 | Pending |
| VDB-02 | Phase 13 | Pending |
| VDB-03 | Phase 13 | Pending |
| VDB-04 | Phase 13 | Pending |
| VDB-05 | Phase 13 | Pending |
| VDB-06 | Phase 13 | Pending |
| VDB-07 | Phase 13 | Pending |
| VDB-08 | Phase 13 | Pending |
| **Synthesis** | | |
| SYN-01 | Phase 14 | Pending |
| SYN-02 | Phase 14 | Pending |
| SYN-03 | Phase 14 | Pending |
| SYN-04 | Phase 14 | Pending |
| SYN-05 | Phase 14 | Pending |
| SYN-06 | Phase 14 | Pending |
| SYN-07 | Phase 14 | Pending |
| SYN-08 | Phase 14 | Pending |
| SYN-09 | Phase 14 | Pending |
| SYN-10 | Phase 14 | Pending |
| **API Endpoints** | | |
| API-01 | Phase 6 | Pending |
| API-02 | Phase 6 | Pending |
| API-03 | Phase 6 | Pending |
| API-04 | Phase 6 | Pending |
| API-05 | Phase 6 | Pending |
| API-06 | Phase 14 | Pending |
| API-07 | Phase 14 | Pending |
| API-08 | Phase 14 | Pending |
| API-09 | Phase 6 | Pending |
| **System Health** | | |
| HLT-01 | Phase 6 | Pending |
| HLT-02 | Phase 6 | Pending |
| HLT-03 | Phase 6 | Pending |
| HLT-04 | Phase 6 | Pending |
| HLT-05 | Phase 6 | Pending |
| **Performance NFRs** | | |
| NFR-PERF-01 | Phase 15 | Pending |
| NFR-PERF-02 | Phase 15 | Pending |
| NFR-PERF-03 | Phase 14 | Pending |
| NFR-PERF-04 | Phase 15 | Pending |
| NFR-PERF-05 | Phase 15 | Pending |
| **Scalability NFRs** | | |
| NFR-SCALE-01 | Phase 13 | Pending |
| NFR-SCALE-02 | Phase 13 | Pending |
| NFR-SCALE-03 | Phase 13 | Pending |
| NFR-SCALE-04 | Phase 13 | Pending |
| **Security NFRs** | | |
| NFR-SEC-01 | Phase 15 | Pending |
| NFR-SEC-02 | Phase 15 | Pending |
| NFR-SEC-03 | Phase 15 | Pending |
| NFR-SEC-04 | Phase 15 | Pending |
| **Reliability NFRs** | | |
| NFR-REL-01 | Phase 15 | Pending |
| NFR-REL-02 | Phase 15 | Pending |
| NFR-REL-03 | Phase 15 | Pending |
| NFR-REL-04 | Phase 15 | Pending |
| **Maintainability NFRs** | | |
| NFR-MAINT-01 | Phase 15 | Pending |
| NFR-MAINT-02 | Phase 15 | Pending |
| NFR-MAINT-03 | Phase 15 | Pending |
| NFR-MAINT-04 | Phase 15 | Pending |

**Coverage:** 120/120 v1 requirements mapped ✓

---

*Requirements document created: 2026-03-27*
*All requirements map to phases 6-15 in ROADMAP.md*
