# Hucki - Production-Grade IRCA RAG Engine

**Status:** Active Development
**Start Date:** 2026-03-27
**Version:** 2.0 (Feature Parity with AnythingLLM)
**Core Value:** Production-grade document processing with zero-cost Claude Code synthesis

---

## What This Is

Hucki is a **production-grade RAG engine** with AnythingLLM-level document processing capabilities, purpose-built for the IRCA workflow (Ideate → Research → Context → Action). It processes **50+ file formats**, scrapes **web/YouTube/GitHub/Confluence**, transcribes **audio**, performs **OCR**, and synthesizes insights using **Claude Code CLI** — all at **zero API cost**.

**Not a lightweight prototype. A feature-complete production system.**

## The Problem

Rob needs to process diverse research inputs for IRCA workflows:
- Research papers (PDF)
- Web documentation (scraping)
- YouTube tutorials (transcript extraction)
- GitHub repositories (code + docs)
- Audio interviews (transcription)
- Scanned documents (OCR)
- Confluence wikis
- 50+ code languages

**Existing solutions fail:**
- **AnythingLLM:** Full-featured but uses Claude API ($15-50/month)
- **NotebookLLM:** Google-hosted, security risks, no API
- **Open WebUI:** Limited format support
- **Custom scripts:** Require constant maintenance

**What's missing:** Production-grade document processing with Claude Code synthesis (zero cost).

## Core Value

**The ONE thing that must work:**
Process 50+ file formats, scrape 4 data connectors (web, YouTube, GitHub, Confluence), transcribe audio, perform OCR, and synthesize high-quality context using Claude Code CLI — with zero external API costs.

If document processing fails or synthesis quality is poor, nothing else matters.

## Context

### Primary User
**Billi** (AI agent) - Orchestrates IRCA workflows, commissions research synthesis

### Secondary User
**Rob** (human) - Benefits from automated multi-source research synthesis

### IRCA Workflow
```
Plane (ideate)
  → n8n (research gathering)
    → Hucki (process + synthesize)
      → context.md
        → Billi/Claudi (execute)
```

### Use Cases

**1. Multi-Format Research Synthesis**
- Input: PDF papers, web docs, YouTube tutorials, GitHub repos, audio interviews
- Hucki: Processes all formats, embeds in Convex, synthesizes via Claude Code
- Output: Structured context.md with citations
- Time: <2 minutes for 10-20 sources

**2. Polymarket Trading Research**
- Sources: PDF whitepaper, API docs (web scrape), CTO podcast (audio), smart contracts (GitHub)
- Hucki: Transcribes audio, clones repo, scrapes docs, synthesizes strategy
- Output: Arbitrage analysis with code examples
- Use: Billi commissions trading bot from synthesis

**3. Next.js Migration Guide**
- Sources: GitHub official repo, blog posts (web), tutorial videos (YouTube), existing codebase (GitHub)
- Hucki: Clones repos, scrapes blogs, extracts transcripts, synthesizes migration plan
- Output: Step-by-step guide with code diffs
- Use: Claudi executes migration

**4. Confluence Wiki Knowledge Base**
- Sources: 20+ Confluence pages, attached PDFs, spreadsheets
- Hucki: Scrapes wiki hierarchy, processes attachments, synthesizes runbook
- Output: On-call playbook with incident history
- Use: Operational documentation

## Tech Decisions

| Decision | Choice | Rationale | Status |
|----------|--------|-----------|--------|
| **Vector DB** | Convex | Already deployed, native vector support, simple API | Validated |
| **LLM** | Claude Code CLI | Zero cost (subscription), no per-token charges | Validated |
| **Embeddings** | sentence-transformers (local) | Free, 384-dim, good quality | Validated |
| **Document Processing** | 6 specialized processors | Parity with AnythingLLM, proven libraries | Pending |
| **Web Scraping** | Puppeteer 21+ | JavaScript rendering, stealth mode, proven | Pending |
| **YouTube** | youtubei.js 9+ | No API key, reliable transcript extraction | Pending |
| **GitHub** | git clone + API | Public/private repos, OAuth support | Pending |
| **Confluence** | REST API | Cloud compatible, attachment support | Pending |
| **Audio Transcription** | Whisper (local + cloud) | Local = free, cloud = fallback for quality | Pending |
| **OCR** | tesseract.js | Multi-language, proven accuracy | Pending |
| **API Framework** | Express.js | Lightweight, n8n-friendly, familiar | Validated |
| **Code Constraint** | <5000 LOC | 10x simpler than AnythingLLM (50k), forces focus | Active |

## Current Milestone: v1.0 Comprehensive

**Goal:** Ship production-ready Hucki with full AnythingLLM feature parity

**Target features:**
- 50+ file format processing (PDFs, DOCX, PPTX, XLSX, 50+ code languages, archives)
- 4 data connectors (web scraping, YouTube, GitHub, Confluence)
- Audio transcription (local Whisper + cloud fallback)
- Image OCR (tesseract.js)
- Video processing (FFmpeg audio extraction)
- Intelligent chunking (sentence boundaries, paragraph-aware)
- Local embeddings (all-MiniLM-L6-v2, 384-dim)
- Convex vector database (workspace isolation, cosine similarity)
- Claude Code CLI synthesis (4 modes: summarize, FAQ, insights, briefing)
- Production-ready (tests, evals, docs, deployment)

**Key constraint:** <5000 LOC (vs AnythingLLM's 50k LOC)

## Requirements

### Active Requirements (v1.0)

#### Workspace Management (WS)
- [ ] **WS-01**: Create workspace with unique slugified ID
- [ ] **WS-02**: List all workspaces with metadata
- [ ] **WS-03**: Delete workspace and all vectors
- [ ] **WS-04**: Get workspace stats (doc_count, vector_count)

#### Document Processing: Text (DOC-TXT)
- [ ] **DOC-TXT-01**: Process PDF files (pdf-parse)
- [ ] **DOC-TXT-02**: Process Word documents (mammoth)
- [ ] **DOC-TXT-03**: Process PowerPoint presentations (officeparser)
- [ ] **DOC-TXT-04**: Process Excel spreadsheets (officeparser)
- [ ] **DOC-TXT-05**: Process plain text files
- [ ] **DOC-TXT-06**: Process Markdown files (marked.js)
- [ ] **DOC-TXT-07**: Process JSON files (JSON.parse)
- [ ] **DOC-TXT-08**: Process CSV files (csv-parse)

#### Document Processing: Code (DOC-CODE)
- [ ] **DOC-CODE-01**: Syntax-aware parsing for 50+ languages
- [ ] **DOC-CODE-02**: Preserve code structure (AST parsing)
- [ ] **DOC-CODE-03**: Extract docstrings and comments
- [ ] **DOC-CODE-04**: Detect imports and dependencies
- [ ] **DOC-CODE-05**: Support web languages (JS, TS, JSX, TSX, HTML, CSS)
- [ ] **DOC-CODE-06**: Support backend languages (Python, Java, C++, Go, Rust, etc.)
- [ ] **DOC-CODE-07**: Support data languages (SQL, R, Julia)
- [ ] **DOC-CODE-08**: Support config formats (YAML, TOML, INI, XML)
- [ ] **DOC-CODE-09**: Support shell scripts (Bash, Zsh, PowerShell)

#### Document Processing: Web (DOC-WEB)
- [ ] **DOC-WEB-01**: Parse HTML files (cheerio)
- [ ] **DOC-WEB-02**: Parse XML files (xml2js)
- [ ] **DOC-WEB-03**: Extract metadata (Open Graph, Twitter Cards)

#### Document Processing: Media (DOC-MEDIA)
- [ ] **DOC-MEDIA-01**: OCR images (PNG, JPG, GIF, BMP, TIFF via tesseract.js)
- [ ] **DOC-MEDIA-02**: Transcribe audio locally (MP3, WAV, M4A via Whisper)
- [ ] **DOC-MEDIA-03**: Transcribe audio via cloud (OpenAI Whisper API fallback)
- [ ] **DOC-MEDIA-04**: Extract audio from video (FFmpeg)
- [ ] **DOC-MEDIA-05**: OCR video frames (FFmpeg + tesseract)

#### Document Processing: Archives (DOC-ARCH)
- [ ] **DOC-ARCH-01**: Extract ZIP archives (adm-zip)
- [ ] **DOC-ARCH-02**: Extract TAR archives (tar-stream)
- [ ] **DOC-ARCH-03**: Recursive extraction (nested archives, 3 levels)

#### Data Connectors: Web Scraping (CONN-WEB)
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

#### Data Connectors: YouTube (CONN-YT)
- [ ] **CONN-YT-01**: Extract transcripts (youtubei.js)
- [ ] **CONN-YT-02**: Multi-language support (auto-detect)
- [ ] **CONN-YT-03**: Timestamp mapping (preserve video timestamps)
- [ ] **CONN-YT-04**: Fallback to Whisper (if no transcript, download audio)

#### Data Connectors: GitHub (CONN-GH)
- [ ] **CONN-GH-01**: Clone repositories (git clone or GitHub API)
- [ ] **CONN-GH-02**: Select branches (default or user-specified)
- [ ] **CONN-GH-03**: Process code + docs (README, docs/, code files)
- [ ] **CONN-GH-04**: Extract issues/PRs (GitHub API)
- [ ] **CONN-GH-05**: OAuth support (GitHub token)

#### Data Connectors: Confluence (CONN-CONF)
- [ ] **CONN-CONF-01**: Scrape wiki pages (Confluence REST API)
- [ ] **CONN-CONF-02**: Preserve hierarchy (parent/child pages)
- [ ] **CONN-CONF-03**: Download attachments (API attachment endpoints)
- [ ] **CONN-CONF-04**: Authentication (API token or OAuth)

#### Data Connectors: Generic (CONN-GEN)
- [ ] **CONN-GEN-01**: Follow redirects (3xx handling)
- [ ] **CONN-GEN-02**: Parse RSS feeds (rss-parser)
- [ ] **CONN-GEN-03**: Extract article content (readability.js)

#### Text Processing & Chunking (CHUNK)
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

#### Embedding Pipeline (EMB)
- [ ] **EMB-01**: Local embeddings (all-MiniLM-L6-v2, 384-dim)
- [ ] **EMB-02**: Alternative model 1 (bge-small-en-v1.5)
- [ ] **EMB-03**: Alternative model 2 (e5-large, 1024-dim)
- [ ] **EMB-04**: Cloud fallback (OpenAI text-embedding-3-small)
- [ ] **EMB-05**: Batch processing (100+ docs simultaneously)
- [ ] **EMB-06**: Caching (skip re-embedding unchanged docs)
- [ ] **EMB-07**: Metadata embedding (titles/headers separately)
- [ ] **EMB-08**: L2 normalization (cosine similarity)

#### Vector Database: Convex (VDB)
- [ ] **VDB-01**: Convex schema (workspaces + vectors tables)
- [ ] **VDB-02**: Vector index (384-dim or 1024-dim)
- [ ] **VDB-03**: Workspace isolation (filter by workspace_id)
- [ ] **VDB-04**: Batch insert (100+ vectors)
- [ ] **VDB-05**: Vector search (cosine similarity, top-k)
- [ ] **VDB-06**: Delete vectors (by workspace_id or vector ID)
- [ ] **VDB-07**: Update vectors (replace existing)
- [ ] **VDB-08**: Namespace management (create/delete workspaces)

#### Synthesis Engine: Claude Code (SYN)
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

#### API Endpoints (API)
- [ ] **API-01**: POST /workspace/create (create workspace)
- [ ] **API-02**: GET /workspaces (list workspaces)
- [ ] **API-03**: GET /workspace/:slug (get workspace)
- [ ] **API-04**: DELETE /workspace/:slug (delete workspace)
- [ ] **API-05**: PATCH /workspace/:slug (update metadata)
- [ ] **API-06**: POST /workspace/:slug/upload (upload document)
- [ ] **API-07**: POST /workspace/:slug/scrape (scrape URL)
- [ ] **API-08**: POST /workspace/:slug/query (query and synthesize)
- [ ] **API-09**: GET /health (health check)

#### System Health (HLT)
- [ ] **HLT-01**: Health check endpoint
- [ ] **HLT-02**: Convex connection status
- [ ] **HLT-03**: Claude Code CLI availability
- [ ] **HLT-04**: Puppeteer status
- [ ] **HLT-05**: System metrics (workspaces, vectors)

#### Non-Functional Requirements (NFR)
- [ ] **NFR-PERF-01**: Document processing <2s for 10-page PDF
- [ ] **NFR-PERF-02**: Vector search <100ms (p95)
- [ ] **NFR-PERF-03**: Synthesis <30s for 10k tokens context
- [ ] **NFR-PERF-04**: API response <500ms (p95, non-synthesis)
- [ ] **NFR-PERF-05**: Concurrent uploads (10 simultaneous)
- [ ] **NFR-SCALE-01**: Vectors per workspace (100k+)
- [ ] **NFR-SCALE-02**: Total workspaces (1000+)
- [ ] **NFR-SCALE-03**: Concurrent workers (10)
- [ ] **NFR-SCALE-04**: Max file size (100MB)
- [ ] **NFR-SEC-01**: No local IP scraping (ALLOW_LOCAL_IPS=false)
- [ ] **NFR-SEC-02**: API key security (store in Billi Vault)
- [ ] **NFR-SEC-03**: Workspace isolation (Convex RLS)
- [ ] **NFR-SEC-04**: Rate limiting (100 req/min per workspace)
- [ ] **NFR-REL-01**: Uptime (99%)
- [ ] **NFR-REL-02**: Error handling (graceful degradation)
- [ ] **NFR-REL-03**: Retry logic (3 attempts max)
- [ ] **NFR-REL-04**: Logging (Winston, structured logs)
- [ ] **NFR-MAINT-01**: Code size (<5000 LOC)
- [ ] **NFR-MAINT-02**: Test coverage (>80%)
- [ ] **NFR-MAINT-03**: Documentation (README + API docs)
- [ ] **NFR-MAINT-04**: Dependencies (<50 npm packages)

**Total Active Requirements:** 120+

### Validated Requirements
(None yet — ship to validate)

### Out of Scope (v1)

#### Explicitly Excluded
- ❌ **Multi-user auth** — Single-user (Rob) for v1
- ❌ **Frontend UI** — API-only, n8n is the UI
- ❌ **Chat interface** — Pure synthesis (no conversational back-and-forth)
- ❌ **Agent system** — No function calling, tool use
- ❌ **Reranking** — Simple cosine similarity (add v2 if needed)
- ❌ **Streaming responses** — Batch synthesis only
- ❌ **Multiple LLM providers** — Claude Code only (core value prop)
- ❌ **Alternative vector DBs** — Convex only (already deployed)

#### Deferred to v2
- 🔮 **Slack connector** — If Rob needs Slack research
- 🔮 **Google Drive connector** — If Rob needs Drive docs
- 🔮 **Notion connector** — If Rob uses Notion
- 🔮 **Advanced reranking** — MMR, Cohere rerank
- 🔮 **Multi-user support** — Team features
- 🔮 **Streaming synthesis** — Real-time output

**Rationale:** Ship comprehensive v1.0 with proven AnythingLLM patterns. Add enterprise features only if validated demand.

## Success Criteria

**Ship v1.0 when:**
1. ✅ All 120+ Active requirements pass integration tests
2. ✅ Quality eval >80% (blind test across 5 scenarios)
3. ✅ 5/5 real-world IRCA workflows complete successfully
4. ✅ Performance targets met (PDF <2s, search <100ms, synthesis <30s)
5. ✅ Code <5000 LOC (excluding tests)
6. ✅ Security passes (auth, isolation, no local IP scraping)
7. ✅ All file formats tested (50+ formats, 4 connectors, audio, OCR)

**Don't ship if:**
- Document processing fails for any P0 format
- Context quality <80% (defeats core value)
- Claude Code CLI unreliable
- Convex integration flaky
- Performance targets missed

## Constraints

### Must Have
- **Cost:** $0 API costs for synthesis (Claude Code, not API)
- **Security:** All data in Convex or local (no external services except cloud Whisper fallback)
- **Speed:** <30s synthesis, <2s document processing
- **Simplicity:** <5000 LOC (10x smaller than AnythingLLM)
- **Feature Parity:** 50+ formats, 4 connectors, audio, OCR (match AnythingLLM)

### Nice to Have
- Comprehensive documentation
- Example n8n workflows
- Performance benchmarks
- Docker deployment

### Out of Bounds
- Don't build a UI (API-only)
- Don't add multi-user features (single-tenant)
- Don't optimize for features not in Active requirements
- Don't exceed 5000 LOC constraint

## Key Decisions

| Date | Decision | Rationale | Outcome |
|------|----------|-----------|---------|
| 2026-03-27 | Convex over LanceDB | Already deployed, simpler integration | Validated |
| 2026-03-27 | Claude Code CLI over API | Zero cost, uses subscription | Validated |
| 2026-03-27 | sentence-transformers (local) | Free, good quality, 384-dim | Validated |
| 2026-03-27 | Full AnythingLLM parity | User directive: no justifications, comprehensive | Active |
| 2026-03-27 | <5000 LOC constraint | Force simplicity (10x smaller than AnythingLLM) | Active |
| 2026-03-27 | Puppeteer for web scraping | Proven, handles SPA, stealth mode | Pending |
| 2026-03-27 | youtubei.js for YouTube | No API key, reliable transcripts | Pending |
| 2026-03-27 | Local Whisper + cloud fallback | Free local, cloud for quality | Pending |
| 2026-03-27 | tesseract.js for OCR | Multi-language, proven | Pending |

## Open Questions

1. **Convex performance at scale:** How does it handle 100k+ vectors per workspace?
2. **Claude Code concurrency:** Can it handle multiple synthesis calls simultaneously?
3. **Puppeteer bot detection:** How often do sites block headless browsers?
4. **Whisper local performance:** Is CPU transcription fast enough or need GPU?
5. **Code size feasibility:** Can we achieve 120+ requirements in <5000 LOC?

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

*Last updated: 2026-03-27 — comprehensive v1.0 scope defined*
