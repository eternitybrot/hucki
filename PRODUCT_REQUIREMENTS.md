# Hucki - Product Requirements Document (PRD)

**Project:** Hucki - IRCA RAG Engine
**Version:** 2.0 (Feature Parity with AnythingLLM Collector)
**Date:** 2026-03-27
**Author:** Claudi (commissioned by Billi)

---

## Executive Summary

**Hucki** is a production-grade RAG (Retrieval-Augmented Generation) engine purpose-built for the IRCA (Ideate → Research → Context → Action) framework. Unlike AnythingLLM which uses expensive Cloud APIs, Hucki uses **Claude Code CLI** for synthesis at **zero API cost**.

**Core Value Proposition:**
Feature-complete document processing and RAG synthesis using local resources + Claude Code subscription (no per-token API charges).

**Key Differentiators:**
- ✅ **50+ file formats** (parity with AnythingLLM)
- ✅ **Web scraping, YouTube, GitHub, Confluence** connectors
- ✅ **Audio transcription** (local Whisper + cloud fallback)
- ✅ **Image OCR** (tesseract.js)
- ✅ **Claude Code CLI synthesis** (not Claude API)
- ✅ **Convex vector database** (not LanceDB)
- ✅ **Zero external API costs** for synthesis
- ✅ **<5000 LOC** (vs AnythingLLM's 50k LOC)

---

## Product Vision

### Problem Statement
Researchers need to process diverse document types (PDFs, audio, web pages, code repos) and synthesize insights. Existing solutions:
- **AnythingLLM:** Full-featured but uses expensive Claude API ($15-50/month)
- **NotebookLM:** Google-hosted, cookie extraction security risks
- **Open WebUI:** Limited file format support
- **Custom scripts:** Require constant maintenance for new formats

### Solution
Hucki provides AnythingLLM-level document processing with Claude Code CLI synthesis:
- Process 50+ file types locally (no cloud upload)
- Scrape web, YouTube, GitHub, Confluence
- Generate embeddings locally (sentence-transformers)
- Store vectors in Convex (already deployed)
- Synthesize with Claude Code CLI (uses Rob's subscription)
- Total cost: $0/month for synthesis

---

## Functional Requirements

### FR-1: Document Processing (50+ Formats)

#### FR-1.1: Text Documents
| ID | Requirement | Format | Implementation | Priority |
|----|-------------|--------|----------------|----------|
| FR-1.1.1 | Process PDF files | .pdf | pdf-parse | P0 |
| FR-1.1.2 | Process Word documents | .docx | mammoth | P0 |
| FR-1.1.3 | Process PowerPoint | .pptx | officeparser | P1 |
| FR-1.1.4 | Process Excel spreadsheets | .xlsx | officeparser | P1 |
| FR-1.1.5 | Process plain text | .txt | fs.readFile | P0 |
| FR-1.1.6 | Process Markdown | .md | marked.js | P0 |
| FR-1.1.7 | Process JSON | .json | JSON.parse | P1 |
| FR-1.1.8 | Process CSV | .csv | csv-parse | P1 |

**Acceptance Criteria:**
- Extract text from each format with <5% data loss
- Preserve metadata (author, date, title)
- Handle malformed files gracefully (return error, don't crash)
- Process 10-page PDF in <2 seconds

#### FR-1.2: Code Files (50+ Languages)
| ID | Requirement | Languages | Implementation | Priority |
|----|-------------|-----------|----------------|----------|
| FR-1.2.1 | Syntax-aware parsing | JavaScript, TypeScript, Python, Java, C++, Go, Rust, PHP, Ruby, Swift | Language-specific tokenizers | P1 |
| FR-1.2.2 | Preserve code structure | All supported | AST parsing (esprima, acorn, tree-sitter) | P1 |
| FR-1.2.3 | Extract docstrings/comments | All supported | Comment parsers | P2 |
| FR-1.2.4 | Detect imports/dependencies | All supported | Dependency graph | P2 |

**Supported Languages (50+):**
- **Web:** JavaScript, TypeScript, JSX, TSX, HTML, CSS, SCSS, Less
- **Backend:** Python, Java, C++, C#, Go, Rust, PHP, Ruby, Kotlin, Swift
- **Data:** SQL, R, Julia, MATLAB
- **Config:** YAML, TOML, INI, XML, JSON
- **Shell:** Bash, Zsh, PowerShell
- **Other:** Solidity, Dart, Elixir, Clojure, Scala

**Acceptance Criteria:**
- Correctly tokenize code files (functions, classes, imports)
- Preserve indentation and formatting
- Extract function signatures and docstrings
- Process 1000-line code file in <500ms

#### FR-1.3: Web Formats
| ID | Requirement | Format | Implementation | Priority |
|----|-------------|--------|----------------|----------|
| FR-1.3.1 | Parse HTML | .html | cheerio | P0 |
| FR-1.3.2 | Parse XML | .xml | xml2js | P1 |
| FR-1.3.3 | Extract metadata | HTML | meta tags, Open Graph, Twitter Cards | P1 |

**Acceptance Criteria:**
- Strip HTML tags, preserve text content
- Extract links and preserve for citation
- Parse XML schema correctly

#### FR-1.4: Media Files
| ID | Requirement | Format | Implementation | Priority |
|----|-------------|--------|----------------|----------|
| FR-1.4.1 | OCR images | .png, .jpg, .gif, .bmp, .tiff | tesseract.js | P0 |
| FR-1.4.2 | Transcribe audio (local) | .mp3, .wav, .m4a, .flac, .ogg | @xenova/transformers (Whisper base) | P0 |
| FR-1.4.3 | Transcribe audio (cloud) | .mp3, .wav, .m4a | OpenAI Whisper API | P1 |
| FR-1.4.4 | Extract audio from video | .mp4, .avi, .mov, .mkv | FFmpeg | P1 |
| FR-1.4.5 | OCR video frames | .mp4, .avi | FFmpeg + tesseract.js | P2 |

**Acceptance Criteria:**
- OCR accuracy >90% for clean text
- Audio transcription: <10% WER (Word Error Rate) for clear speech
- Local transcription: 1 minute audio processes in <10 seconds
- Cloud transcription: 1 minute audio processes in <2 seconds
- Handle multi-language audio (detect language automatically)

#### FR-1.5: Archives
| ID | Requirement | Format | Implementation | Priority |
|----|-------------|--------|----------------|----------|
| FR-1.5.1 | Extract ZIP archives | .zip | adm-zip | P1 |
| FR-1.5.2 | Extract TAR archives | .tar, .tar.gz, .tgz | tar-stream | P2 |
| FR-1.5.3 | Recursive extraction | Nested archives | Recursive processor | P2 |

**Acceptance Criteria:**
- Extract all files from archive
- Process extracted files through appropriate processors
- Handle nested archives (up to 3 levels deep)

---

### FR-2: Data Connector System

#### FR-2.1: Web Scraping
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-2.1.1 | Scrape single URL | Puppeteer 21+ | P0 |
| FR-2.1.2 | Render JavaScript | Puppeteer headless Chrome | P0 |
| FR-2.1.3 | Wait for dynamic content | networkidle0, domcontentloaded | P0 |
| FR-2.1.4 | Custom headers | User-Agent, cookies | P1 |
| FR-2.1.5 | Sitemap crawling | Parse sitemap.xml | P1 |
| FR-2.1.6 | Recursive crawling | Depth-limited, same-domain | P2 |
| FR-2.1.7 | Respect robots.txt | robots-parser | P1 |
| FR-2.1.8 | Rate limiting | Configurable delay (default 1s) | P1 |
| FR-2.1.9 | Screenshot capture | page.screenshot() | P2 |
| FR-2.1.10 | Proxy support | HTTP/SOCKS proxies | P2 |

**Acceptance Criteria:**
- Successfully scrape 95% of public websites
- Handle SPA (React, Vue, Angular) correctly
- Respect rate limits (default 1 req/sec)
- Block local IP scraping by default (`ALLOW_LOCAL_IPS=false`)

#### FR-2.2: YouTube Integration
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-2.2.1 | Extract transcripts | youtubei.js 9+ | P0 |
| FR-2.2.2 | Multi-language support | Auto-detect language | P1 |
| FR-2.2.3 | Timestamp mapping | Preserve video timestamps | P1 |
| FR-2.2.4 | Fallback to Whisper | If no transcript available, download audio + transcribe | P2 |

**Acceptance Criteria:**
- Extract transcript from 95% of public videos
- Process 10-minute video transcript in <5 seconds
- Preserve timestamps for citation

#### FR-2.3: GitHub Integration
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-2.3.1 | Clone repositories | git clone or GitHub API | P0 |
| FR-2.3.2 | Select branches | Default branch or user-specified | P1 |
| FR-2.3.3 | Process code + docs | README, docs/, code files | P0 |
| FR-2.3.4 | Extract issues/PRs | GitHub API | P2 |
| FR-2.3.5 | OAuth support | GitHub OAuth token | P1 |

**Acceptance Criteria:**
- Clone public repos without auth
- Clone private repos with OAuth token
- Process repos up to 100MB (skip large binaries)
- Extract README, docs, code files

#### FR-2.4: Confluence Integration
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-2.4.1 | Scrape wiki pages | Confluence REST API | P1 |
| FR-2.4.2 | Preserve hierarchy | Parent/child pages | P2 |
| FR-2.4.3 | Download attachments | API attachment endpoints | P2 |
| FR-2.4.4 | Authentication | API token or OAuth | P1 |

**Acceptance Criteria:**
- Scrape Confluence Cloud pages
- Preserve page hierarchy (breadcrumbs)
- Download and process attachments (PDFs, images)

#### FR-2.5: Generic URL Connector
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-2.5.1 | Follow redirects | Puppeteer or axios | P0 |
| FR-2.5.2 | Parse RSS feeds | rss-parser | P1 |
| FR-2.5.3 | Extract article content | readability.js | P1 |

**Acceptance Criteria:**
- Handle 3xx redirects correctly
- Parse RSS/Atom feeds
- Extract main article content (strip nav, ads, footers)

---

### FR-3: Text Processing & Chunking

#### FR-3.1: Chunking Strategy
| ID | Requirement | Default | Range | Priority |
|----|-------------|---------|-------|----------|
| FR-3.1.1 | Configurable chunk size | 1000 tokens | 100-5000 | P0 |
| FR-3.1.2 | Configurable overlap | 20 tokens | 0-500 | P0 |
| FR-3.1.3 | Sentence boundary preservation | Enabled | - | P0 |
| FR-3.1.4 | Paragraph-aware chunking | Enabled | - | P1 |
| FR-3.1.5 | Min chunk size | 100 tokens | 50-500 | P1 |

**Acceptance Criteria:**
- No mid-sentence splits (unless sentence >1000 tokens)
- Preserve paragraph boundaries when possible
- Handle edge cases (lists, code blocks, tables)

#### FR-3.2: Preprocessing
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-3.2.1 | Whitespace normalization | Collapse multiple spaces, trim lines | P0 |
| FR-3.2.2 | Language detection | franc or langdetect | P1 |
| FR-3.2.3 | Markdown preservation | Keep headings, lists, code blocks | P0 |
| FR-3.2.4 | Code syntax preservation | Maintain indentation, syntax | P1 |
| FR-3.2.5 | Link extraction | Separate inline links, preserve for citations | P1 |
| FR-3.2.6 | Metadata extraction | Title, author, date, source URL | P0 |

**Acceptance Criteria:**
- Clean text without mangling markdown
- Preserve code formatting
- Extract all metadata fields when available

---

### FR-4: Embedding Pipeline

#### FR-4.1: Local Embeddings
| ID | Requirement | Model | Dimensions | Priority |
|----|-------------|-------|------------|----------|
| FR-4.1.1 | Default local embedder | all-MiniLM-L6-v2 | 384 | P0 |
| FR-4.1.2 | Alternative model 1 | bge-small-en-v1.5 | 384 | P1 |
| FR-4.1.3 | Alternative model 2 | e5-large | 1024 | P2 |
| FR-4.1.4 | Implementation | @xenova/transformers | - | P0 |

**Acceptance Criteria:**
- Process 100 docs/sec on CPU
- Generate 384-dim vectors
- L2 normalization for cosine similarity
- Cache embeddings (don't re-embed unchanged docs)

#### FR-4.2: Cloud Embeddings (Fallback)
| ID | Requirement | Provider | Priority |
|----|-------------|----------|----------|
| FR-4.2.1 | OpenAI embeddings | text-embedding-3-small | P2 |
| FR-4.2.2 | Cohere embeddings | embed-english-v3.0 | P2 |
| FR-4.2.3 | Voyage AI embeddings | voyage-2 | P2 |

**Note:** Cloud embeddings are **fallback only** when local fails or for higher quality needs.

**Acceptance Criteria:**
- Fallback to cloud if local model fails
- API key stored in Billi Vault
- Track API usage costs

#### FR-4.3: Embedding Features
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-4.3.1 | Batch processing | Process 100+ docs simultaneously | P0 |
| FR-4.3.2 | Caching | Skip re-embedding for unchanged docs | P0 |
| FR-4.3.3 | Metadata embedding | Optionally embed titles/headers separately | P1 |

**Acceptance Criteria:**
- Cache hit rate >80% for repeat docs
- Batch API calls (reduce overhead)

---

### FR-5: Vector Database (Convex)

#### FR-5.1: Convex Schema
```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  workspaces: defineTable({
    name: v.string(),
    slug: v.string(),
    created_at: v.number(),
    doc_count: v.number(),
    metadata: v.optional(v.object({
      description: v.string(),
      tags: v.array(v.string()),
    })),
  }).index("by_slug", ["slug"]),

  vectors: defineTable({
    workspace_id: v.id("workspaces"),
    embedding: v.array(v.float64()),
    text: v.string(),
    metadata: v.object({
      source: v.string(),
      chunk_index: v.number(),
      doc_type: v.string(),
      created_at: v.number(),
    }),
  })
  .vectorIndex("by_embedding", {
    vectorField: "embedding",
    dimensions: 384,
    filterFields: ["workspace_id"],
  })
  .index("by_workspace", ["workspace_id"]),
});
```

#### FR-5.2: Vector Operations
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-5.2.1 | Add vectors | Batch insert (100+ vectors) | P0 |
| FR-5.2.2 | Search vectors | Cosine similarity, top-k | P0 |
| FR-5.2.3 | Delete vectors | By workspace_id or vector ID | P1 |
| FR-5.2.4 | Update vectors | Replace existing vectors | P1 |
| FR-5.2.5 | Namespace management | Create/delete workspaces | P0 |

**Acceptance Criteria:**
- Insert 100 vectors in <500ms
- Search returns top-k results (k=4 default)
- Delete workspace removes all vectors
- Vector index supports cosine similarity search
- Filter results by workspace_id
- Handle 10k+ vectors per workspace
- Search latency <100ms (p95)

---

### FR-6: Synthesis Engine (Claude Code CLI)

#### FR-6.1: Claude Code Integration
| ID | Requirement | Implementation | Priority |
|----|-------------|----------------|----------|
| FR-6.1.1 | CLI invocation | `echo "prompt" \| claude --print --no-stream` | P0 |
| FR-6.1.2 | Context injection | Inject top-k retrieved chunks | P0 |
| FR-6.1.3 | Prompt engineering | IRCA-specific prompts | P0 |
| FR-6.1.4 | Output parsing | Extract markdown output | P0 |
| FR-6.1.5 | Error handling | Retry on failure (max 3 attempts) | P1 |

**Acceptance Criteria:**
- Successfully invoke Claude Code CLI
- No API token required (uses Rob's credentials)
- Parse Claude output correctly
- Handle rate limits gracefully

#### FR-6.2: Synthesis Modes
| ID | Requirement | Description | Priority |
|----|-------------|-------------|----------|
| FR-6.2.1 | Summarize | Generate executive summary | P0 |
| FR-6.2.2 | FAQ generation | Extract Q&A pairs | P1 |
| FR-6.2.3 | Key insights | Bullet points of key findings | P0 |
| FR-6.2.4 | Briefing document | Full structured report | P0 |
| FR-6.2.5 | Custom prompt | User-defined synthesis | P1 |

**Acceptance Criteria:**
- Each mode generates appropriate output
- Synthesis completes in <30 seconds (p95)
- Output is markdown-formatted

---

### FR-7: Workspace Management

#### FR-7.1: Workspace Operations
| ID | Requirement | API Endpoint | Priority |
|----|-------------|--------------|----------|
| FR-7.1.1 | Create workspace | `POST /workspace/create` | P0 |
| FR-7.1.2 | List workspaces | `GET /workspaces` | P0 |
| FR-7.1.3 | Get workspace | `GET /workspace/:slug` | P0 |
| FR-7.1.4 | Delete workspace | `DELETE /workspace/:slug` | P1 |
| FR-7.1.5 | Update metadata | `PATCH /workspace/:slug` | P1 |

**Acceptance Criteria:**
- Slug is auto-generated from name (slugify)
- Workspace isolation (no cross-workspace leaks)
- Deleting workspace removes all vectors

---

### FR-8: API Endpoints

#### FR-8.1: Document Upload
```
POST /workspace/:slug/upload
Content-Type: multipart/form-data

Body:
  - file: File (PDF, DOCX, MP3, etc.)
  - metadata: JSON (optional)

Response:
{
  "success": true,
  "data": {
    "doc_id": "...",
    "chunks_created": 42,
    "processing_time_ms": 1234
  }
}
```

#### FR-8.2: URL Processing
```
POST /workspace/:slug/scrape
Content-Type: application/json

Body:
{
  "url": "https://example.com",
  "type": "web" | "youtube" | "github" | "confluence",
  "options": {
    "depth": 1,
    "branch": "main"
  }
}

Response:
{
  "success": true,
  "data": {
    "pages_scraped": 10,
    "chunks_created": 420,
    "processing_time_ms": 5678
  }
}
```

#### FR-8.3: Query Workspace
```
POST /workspace/:slug/query
Content-Type: application/json

Body:
{
  "query": "What are the key findings?",
  "top_k": 10,
  "min_similarity": 0.7,
  "mode": "summarize" | "faq" | "insights" | "briefing" | "custom",
  "custom_prompt": "..."
}

Response:
{
  "success": true,
  "data": {
    "synthesis": "# Summary\n\n...",
    "sources": [
      {"source": "doc.pdf", "page": 1, "chunk_index": 0}
    ],
    "synthesis_time_ms": 15000
  }
}
```

#### FR-8.4: Health Check
```
GET /health

Response:
{
  "status": "healthy",
  "services": {
    "convex": "connected",
    "claude_code": "available",
    "puppeteer": "ready"
  },
  "stats": {
    "total_workspaces": 42,
    "total_vectors": 12345
  }
}
```

---

## Non-Functional Requirements

### NFR-1: Performance
| Metric | Target | Priority |
|--------|--------|----------|
| Document processing | <2s for 10-page PDF | P0 |
| Vector search | <100ms (p95) | P0 |
| Synthesis | <30s for 10k tokens context | P0 |
| API response time | <500ms (p95) | P1 |
| Concurrent uploads | 10 simultaneous | P1 |

### NFR-2: Scalability
| Metric | Target | Priority |
|--------|--------|----------|
| Vectors per workspace | 100k+ | P1 |
| Total workspaces | 1000+ | P1 |
| Concurrent workers | 10 | P1 |
| Max file size | 100MB | P1 |

### NFR-3: Security
| Requirement | Implementation | Priority |
|-------------|----------------|----------|
| No local IP scraping | `ALLOW_LOCAL_IPS=false` default | P0 |
| API key security | Store in Billi Vault | P0 |
| Workspace isolation | Convex RLS (row-level security) | P0 |
| Rate limiting | 100 req/min per workspace | P1 |

### NFR-4: Reliability
| Requirement | Target | Priority |
|-------------|--------|----------|
| Uptime | 99% | P1 |
| Error handling | Graceful degradation | P0 |
| Retry logic | 3 attempts max | P0 |
| Logging | Winston, structured logs | P1 |

### NFR-5: Maintainability
| Requirement | Target | Priority |
|-------------|--------|----------|
| Code size | <5000 LOC | P0 |
| Test coverage | >80% | P0 |
| Documentation | Comprehensive README + API docs | P1 |
| Dependencies | <50 npm packages | P1 |

---

## Success Metrics

### Metric 1: Feature Parity
**Definition:** Support same file formats and connectors as AnythingLLM
**Target:** 95% parity (48/50 formats)
**Measurement:** Test suite coverage

### Metric 2: Cost Savings
**Definition:** Zero synthesis API costs
**Target:** $0/month (vs $15-50 for AnythingLLM with Claude API)
**Measurement:** No Claude API calls (only Claude Code CLI)

### Metric 3: Performance
**Definition:** Comparable processing speed to AnythingLLM
**Target:**
- PDF processing: <2s for 10 pages
- Vector search: <100ms
- Synthesis: <30s

### Metric 4: Code Simplicity
**Definition:** Minimal codebase for maintainability
**Target:** <5000 LOC (10x smaller than AnythingLLM)
**Measurement:** `cloc` report

---

## Out of Scope (v1)

### Explicitly Excluded
- ❌ **Multi-user auth:** Single-user (Rob) for v1
- ❌ **Frontend UI:** API-only, n8n is the UI
- ❌ **Chat interface:** Pure synthesis (no conversational back-and-forth)
- ❌ **Agent system:** No function calling, tool use
- ❌ **Reranking:** Simple cosine similarity
- ❌ **Streaming responses:** Batch synthesis only
- ❌ **Multiple LLM providers:** Claude Code only
- ❌ **Alternative vector DBs:** Convex only

---

## Dependencies

### System Dependencies
```bash
apt-get install -y ffmpeg tesseract-ocr
```

### NPM Dependencies (~40 packages)
```json
{
  "puppeteer": "^21.5.2",
  "pdf-parse": "^1.1.1",
  "mammoth": "^1.6.0",
  "officeparser": "^4.0.5",
  "tesseract.js": "^5.0.0",
  "youtubei.js": "^9.1.0",
  "@xenova/transformers": "^2.14.0",
  "cheerio": "^1.0.0-rc.12",
  "xml2js": "^0.6.2",
  "csv-parse": "^5.5.3",
  "marked": "^11.1.1",
  "adm-zip": "^0.5.10",
  "sharp": "^0.33.1",
  "express": "^4.18.2",
  "multer": "^1.4.5-lts.1",
  "convex": "^1.5.0",
  "winston": "^3.11.0"
}
```

---

## Risk Assessment

### Risk 1: Claude Code CLI Availability
**Likelihood:** Low
**Impact:** High
**Mitigation:** Fallback to Claude API (with user confirmation + cost warning)

### Risk 2: Local Whisper Performance
**Likelihood:** Medium
**Impact:** Medium
**Mitigation:** Cloud Whisper fallback, optimize with GPU acceleration

### Risk 3: Puppeteer Bot Detection
**Likelihood:** Medium
**Impact:** Low
**Mitigation:** User-Agent rotation, stealth plugin, manual fallback

### Risk 4: Convex Vector Search Performance
**Likelihood:** Low
**Impact:** Medium
**Mitigation:** Test at 100k+ vectors, optimize indexing if needed
