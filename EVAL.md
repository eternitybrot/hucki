# Hucki - Evaluation Specification

**Project:** Hucki - IRCA RAG Engine
**Version:** 2.0 (Comprehensive Test Suite)
**Date:** 2026-03-27
**Author:** Claudi (commissioned by Billi)

---

## Executive Summary

This evaluation specification defines comprehensive testing for **Hucki's 50+ file format support**, **4 data connectors**, **media processing pipeline**, and **Claude Code synthesis**. The eval is designed to be **blind** — implementation details are hidden from the evaluator.

**Evaluation Levels:**
1. **Unit Tests** (automated) — Test individual components in isolation
2. **Integration Tests** (automated) — Test full processing pipeline
3. **Quality Tests** (automated + manual) — Measure synthesis accuracy
4. **Real-World Scenarios** (blind) — End-to-end IRCA workflows

**Pass Criteria:**
- ✅ 100% unit test pass rate
- ✅ 100% integration test pass rate
- ✅ >80% quality score (blind eval)
- ✅ All 5 real-world scenarios complete successfully

---

## Level 1: Unit Tests (Automated)

### 1.1: Document Processors (50+ Formats)

#### Test Suite: Text Documents
```javascript
describe("Document Processors: Text", () => {
  describe("PDF Processor", () => {
    test("extracts text from simple PDF", async () => {
      const result = await processDocument("fixtures/simple.pdf");
      expect(result.text).toContain("Lorem ipsum");
      expect(result.metadata.pages).toBe(3);
      expect(result.processing_time_ms).toBeLessThan(2000);
    });

    test("handles scanned PDF gracefully", async () => {
      const result = await processDocument("fixtures/scanned.pdf");
      expect(result.error).toBeDefined();
      expect(result.error).toContain("OCR not supported");
    });

    test("preserves metadata", async () => {
      const result = await processDocument("fixtures/metadata.pdf");
      expect(result.metadata.author).toBe("John Doe");
      expect(result.metadata.title).toBeDefined();
      expect(result.metadata.created_at).toBeDefined();
    });
  });

  describe("DOCX Processor", () => {
    test("extracts paragraphs correctly", async () => {
      const result = await processDocument("fixtures/document.docx");
      expect(result.text).toContain("Heading 1");
      expect(result.paragraphs.length).toBeGreaterThan(5);
    });

    test("preserves tables", async () => {
      const result = await processDocument("fixtures/table.docx");
      expect(result.tables.length).toBe(1);
      expect(result.tables[0].rows).toBe(5);
    });
  });

  describe("PPTX Processor", () => {
    test("extracts slide text", async () => {
      const result = await processDocument("fixtures/presentation.pptx");
      expect(result.slides.length).toBe(10);
      expect(result.slides[0].title).toBeDefined();
    });

    test("extracts speaker notes", async () => {
      const result = await processDocument("fixtures/notes.pptx");
      expect(result.slides[0].notes).toContain("Remember to");
    });
  });

  describe("XLSX Processor", () => {
    test("extracts all sheets", async () => {
      const result = await processDocument("fixtures/spreadsheet.xlsx");
      expect(result.sheets.length).toBe(3);
      expect(result.sheets[0].name).toBe("Sheet1");
    });

    test("handles formulas", async () => {
      const result = await processDocument("fixtures/formulas.xlsx");
      expect(result.sheets[0].cells['A1']).toBe("100");
      expect(result.sheets[0].cells['B1']).toBe("=SUM(A1:A10)");
    });
  });

  describe("Markdown Processor", () => {
    test("preserves headings and structure", async () => {
      const result = await processDocument("fixtures/structure.md");
      expect(result.headings.length).toBeGreaterThan(0);
      expect(result.code_blocks.length).toBeGreaterThan(0);
    });

    test("extracts links", async () => {
      const result = await processDocument("fixtures/links.md");
      expect(result.links.length).toBeGreaterThan(5);
      expect(result.links[0].text).toBeDefined();
      expect(result.links[0].url).toBeDefined();
    });
  });

  describe("JSON Processor", () => {
    test("parses valid JSON", async () => {
      const result = await processDocument("fixtures/data.json");
      expect(result.parsed).toBeDefined();
      expect(result.schema_detected).toBe(true);
    });

    test("handles malformed JSON gracefully", async () => {
      const result = await processDocument("fixtures/malformed.json");
      expect(result.error).toBeDefined();
      expect(result.error).toContain("JSON parse error");
    });
  });

  describe("CSV Processor", () => {
    test("detects headers", async () => {
      const result = await processDocument("fixtures/data.csv");
      expect(result.headers).toEqual(["name", "age", "email"]);
      expect(result.rows.length).toBe(100);
    });

    test("handles quoted fields", async () => {
      const result = await processDocument("fixtures/quotes.csv");
      expect(result.rows[0][1]).toBe("Smith, John");
    });
  });
});
```

#### Test Suite: Code Files (50+ Languages)
```javascript
describe("Document Processors: Code", () => {
  const testLanguage = (lang, file, expected) => {
    test(`parses ${lang} correctly`, async () => {
      const result = await processDocument(`fixtures/code/${file}`);
      expect(result.language).toBe(lang);
      expect(result.functions.length).toBeGreaterThanOrEqual(expected.functions);
      expect(result.classes.length).toBeGreaterThanOrEqual(expected.classes);
      expect(result.processing_time_ms).toBeLessThan(500);
    });
  };

  testLanguage("javascript", "sample.js", { functions: 5, classes: 2 });
  testLanguage("typescript", "sample.ts", { functions: 3, classes: 1 });
  testLanguage("python", "sample.py", { functions: 8, classes: 3 });
  testLanguage("java", "Sample.java", { functions: 4, classes: 1 });
  testLanguage("cpp", "sample.cpp", { functions: 6, classes: 2 });
  testLanguage("go", "sample.go", { functions: 4, classes: 0 });
  testLanguage("rust", "sample.rs", { functions: 5, classes: 0 });
  testLanguage("php", "sample.php", { functions: 3, classes: 1 });
  testLanguage("ruby", "sample.rb", { functions: 4, classes: 2 });
  testLanguage("swift", "sample.swift", { functions: 3, classes: 1 });

  test("extracts docstrings", async () => {
    const result = await processDocument("fixtures/code/documented.py");
    expect(result.functions[0].docstring).toContain("Calculate");
  });

  test("detects imports", async () => {
    const result = await processDocument("fixtures/code/imports.js");
    expect(result.imports.length).toBeGreaterThan(5);
    expect(result.imports).toContain("react");
  });

  test("preserves indentation", async () => {
    const result = await processDocument("fixtures/code/indented.py");
    expect(result.text).toContain("    def nested():");
  });
});
```

#### Test Suite: Media Files
```javascript
describe("Document Processors: Media", () => {
  describe("Image OCR", () => {
    test("extracts text from clean image", async () => {
      const result = await processDocument("fixtures/media/clean_text.png");
      expect(result.text).toContain("Hello World");
      expect(result.confidence).toBeGreaterThan(0.9);
    });

    test("handles poor quality images", async () => {
      const result = await processDocument("fixtures/media/blurry.jpg");
      expect(result.text).toBeDefined();
      expect(result.confidence).toBeLessThan(0.7);
    });

    test("supports multiple languages", async () => {
      const result = await processDocument("fixtures/media/spanish.png");
      expect(result.language_detected).toBe("spa");
    });
  });

  describe("Audio Transcription (Local)", () => {
    test("transcribes clear speech", async () => {
      const result = await processDocument("fixtures/media/clear_speech.wav");
      expect(result.text).toContain("good morning");
      expect(result.transcription_method).toBe("whisper_local");
      expect(result.processing_time_ms).toBeLessThan(10000); // <10s for 1min audio
    });

    test("handles background noise", async () => {
      const result = await processDocument("fixtures/media/noisy.mp3");
      expect(result.text).toBeDefined();
      expect(result.wer).toBeLessThan(0.2); // <20% Word Error Rate
    });

    test("detects language", async () => {
      const result = await processDocument("fixtures/media/french.m4a");
      expect(result.language_detected).toBe("fr");
    });
  });

  describe("Video Processing", () => {
    test("extracts audio and transcribes", async () => {
      const result = await processDocument("fixtures/media/video.mp4");
      expect(result.audio_extracted).toBe(true);
      expect(result.text).toBeDefined();
    });

    test("handles videos without audio", async () => {
      const result = await processDocument("fixtures/media/silent.mp4");
      expect(result.audio_extracted).toBe(false);
      expect(result.frames_extracted).toBeGreaterThan(0);
    });
  });
});
```

#### Test Suite: Archives
```javascript
describe("Document Processors: Archives", () => {
  test("extracts ZIP archives", async () => {
    const result = await processDocument("fixtures/archive.zip");
    expect(result.files_extracted).toBe(10);
    expect(result.processed_files).toBe(10);
  });

  test("handles nested archives", async () => {
    const result = await processDocument("fixtures/nested.zip");
    expect(result.nesting_level).toBe(2);
    expect(result.files_extracted).toBeGreaterThan(5);
  });

  test("handles corrupted archives gracefully", async () => {
    const result = await processDocument("fixtures/corrupted.zip");
    expect(result.error).toBeDefined();
    expect(result.error).toContain("corrupted");
  });
});
```

### 1.2: Data Connectors

#### Test Suite: Web Scraping
```javascript
describe("Data Connectors: Web Scraping", () => {
  test("scrapes static webpage", async () => {
    const result = await scrapeURL("https://example.com");
    expect(result.text).toBeDefined();
    expect(result.links).toBeInstanceOf(Array);
    expect(result.metadata.title).toBe("Example Domain");
  });

  test("renders JavaScript (SPA)", async () => {
    const result = await scrapeURL("https://spa-example.com");
    expect(result.dynamic_content_rendered).toBe(true);
    expect(result.text).toContain("dynamically loaded");
  });

  test("respects robots.txt", async () => {
    const result = await scrapeURL("https://blocked-site.com/disallowed");
    expect(result.error).toContain("robots.txt");
  });

  test("rate limiting works", async () => {
    const start = Date.now();
    await scrapeURL("https://example.com/page1");
    await scrapeURL("https://example.com/page2");
    const elapsed = Date.now() - start;
    expect(elapsed).toBeGreaterThan(1000); // 1s delay
  });

  test("blocks local IPs by default", async () => {
    const result = await scrapeURL("http://192.168.1.1");
    expect(result.error).toContain("local IP");
  });
});
```

#### Test Suite: YouTube
```javascript
describe("Data Connectors: YouTube", () => {
  test("extracts transcript from public video", async () => {
    const result = await scrapeYouTube("https://youtube.com/watch?v=dQw4w9WgXcQ");
    expect(result.transcript).toBeDefined();
    expect(result.timestamps).toBeInstanceOf(Array);
    expect(result.language).toBe("en");
  });

  test("handles multi-language transcripts", async () => {
    const result = await scrapeYouTube("https://youtube.com/watch?v=multi_lang");
    expect(result.languages_available).toContain("en");
    expect(result.languages_available).toContain("es");
  });

  test("falls back to Whisper if no transcript", async () => {
    const result = await scrapeYouTube("https://youtube.com/watch?v=no_transcript");
    expect(result.transcript_method).toBe("whisper_fallback");
    expect(result.transcript).toBeDefined();
  });
});
```

#### Test Suite: GitHub
```javascript
describe("Data Connectors: GitHub", () => {
  test("clones public repository", async () => {
    const result = await scrapeGitHub("https://github.com/user/repo");
    expect(result.files_processed).toBeGreaterThan(10);
    expect(result.readme_found).toBe(true);
    expect(result.code_files).toBeGreaterThan(5);
  });

  test("selects specific branch", async () => {
    const result = await scrapeGitHub("https://github.com/user/repo", { branch: "dev" });
    expect(result.branch).toBe("dev");
  });

  test("handles large repos (size limit)", async () => {
    const result = await scrapeGitHub("https://github.com/user/huge-repo");
    expect(result.size_limit_exceeded).toBe(true);
    expect(result.files_processed).toBeLessThan(result.total_files);
  });

  test("requires auth for private repos", async () => {
    const result = await scrapeGitHub("https://github.com/user/private-repo");
    expect(result.error).toContain("authentication required");
  });
});
```

#### Test Suite: Confluence
```javascript
describe("Data Connectors: Confluence", () => {
  test("scrapes Confluence page", async () => {
    const result = await scrapeConfluence("https://company.atlassian.net/wiki/...");
    expect(result.text).toBeDefined();
    expect(result.metadata.title).toBeDefined();
  });

  test("preserves page hierarchy", async () => {
    const result = await scrapeConfluence("https://company.atlassian.net/wiki/...");
    expect(result.breadcrumbs).toBeInstanceOf(Array);
    expect(result.parent_page).toBeDefined();
  });

  test("requires authentication", async () => {
    const result = await scrapeConfluence("https://company.atlassian.net/wiki/...", { auth: null });
    expect(result.error).toContain("authentication");
  });
});
```

### 1.3: Chunking & Preprocessing

```javascript
describe("Text Processing: Chunking", () => {
  test("chunks at sentence boundaries", async () => {
    const text = "Sentence one. Sentence two. Sentence three.";
    const chunks = await chunkText(text, { max_size: 20 });
    expect(chunks.length).toBeGreaterThan(1);
    expect(chunks[0]).toMatch(/\.$/); // Ends with period
  });

  test("preserves paragraph boundaries", async () => {
    const text = "Paragraph 1.\n\nParagraph 2.\n\nParagraph 3.";
    const chunks = await chunkText(text, { max_size: 30 });
    expect(chunks[0]).not.toContain("Paragraph 2");
  });

  test("handles code blocks", async () => {
    const text = "Some text\n```python\ncode here\n```\nMore text";
    const chunks = await chunkText(text);
    const codeChunk = chunks.find(c => c.includes("```"));
    expect(codeChunk).toContain("```python");
    expect(codeChunk).toContain("```");
  });

  test("applies overlap correctly", async () => {
    const text = "A B C D E F G H I J K L M N O P";
    const chunks = await chunkText(text, { max_size: 10, overlap: 2 });
    expect(chunks[0].slice(-2)).toBe(chunks[1].slice(0, 2));
  });

  test("normalizes whitespace", async () => {
    const text = "Extra    spaces   here";
    const chunks = await chunkText(text);
    expect(chunks[0]).toBe("Extra spaces here");
  });
});
```

### 1.4: Embedding Pipeline

```javascript
describe("Embedding Pipeline", () => {
  test("generates 384-dim embeddings", async () => {
    const result = await embedText("Sample text");
    expect(result.embedding.length).toBe(384);
    expect(result.embedding[0]).toBeTypeOf("number");
  });

  test("batch processing works", async () => {
    const texts = Array(100).fill("Sample text");
    const start = Date.now();
    const results = await embedBatch(texts);
    const elapsed = Date.now() - start;
    expect(results.length).toBe(100);
    expect(elapsed).toBeLessThan(1000); // <1s for 100 docs
  });

  test("caches embeddings", async () => {
    const text = "Same text";
    await embedText(text);
    const start = Date.now();
    await embedText(text); // Should hit cache
    const elapsed = Date.now() - start;
    expect(elapsed).toBeLessThan(10); // Cache hit <10ms
  });

  test("normalizes vectors", async () => {
    const result = await embedText("Sample");
    const magnitude = Math.sqrt(result.embedding.reduce((sum, x) => sum + x*x, 0));
    expect(magnitude).toBeCloseTo(1.0, 2);
  });
});
```

### 1.5: Vector Database (Convex)

```javascript
describe("Vector Database: Convex", () => {
  test("creates workspace", async () => {
    const ws = await createWorkspace("Test Project");
    expect(ws.id).toBeDefined();
    expect(ws.slug).toBe("test-project");
    expect(ws.doc_count).toBe(0);
  });

  test("inserts vectors in batch", async () => {
    const vectors = Array(100).fill({ embedding: Array(384).fill(0.1), text: "test" });
    const start = Date.now();
    await insertVectors("test-workspace", vectors);
    const elapsed = Date.now() - start;
    expect(elapsed).toBeLessThan(500);
  });

  test("vector search returns top-k", async () => {
    const query = Array(384).fill(0.5);
    const results = await searchVectors("test-workspace", query, { top_k: 5 });
    expect(results.length).toBe(5);
    expect(results[0].score).toBeGreaterThan(results[4].score);
  });

  test("filters by workspace_id", async () => {
    await insertVectors("workspace-1", [{ embedding: [...], text: "ws1" }]);
    await insertVectors("workspace-2", [{ embedding: [...], text: "ws2" }]);
    const results = await searchVectors("workspace-1", query);
    expect(results.every(r => r.text !== "ws2")).toBe(true);
  });

  test("deletes workspace and vectors", async () => {
    await createWorkspace("temp");
    await insertVectors("temp", [{ embedding: [...], text: "test" }]);
    await deleteWorkspace("temp");
    const results = await searchVectors("temp", query);
    expect(results.length).toBe(0);
  });
});
```

### 1.6: Claude Code CLI Integration

```javascript
describe("Synthesis: Claude Code CLI", () => {
  test("invokes Claude Code successfully", async () => {
    const result = await synthesize("Summarize this: Hello world");
    expect(result.output).toBeDefined();
    expect(result.error).toBeUndefined();
  });

  test("handles large context (10k tokens)", async () => {
    const context = "word ".repeat(10000);
    const result = await synthesize(`Summarize: ${context}`);
    expect(result.output).toBeDefined();
    expect(result.processing_time_ms).toBeLessThan(30000);
  });

  test("retries on failure", async () => {
    // Mock failure on first attempt
    mockClaudeFail(1);
    const result = await synthesize("test");
    expect(result.retries).toBe(1);
    expect(result.output).toBeDefined();
  });

  test("returns markdown output", async () => {
    const result = await synthesize("Generate FAQ");
    expect(result.output).toMatch(/^#/); // Starts with heading
    expect(result.format).toBe("markdown");
  });
});
```

---

## Level 2: Integration Tests (Automated)

### 2.1: End-to-End Document Processing

```javascript
describe("Integration: Document Upload → Vectors", () => {
  test("PDF upload pipeline", async () => {
    // 1. Upload PDF
    const upload = await uploadDocument("workspace-1", "research.pdf");
    expect(upload.success).toBe(true);

    // 2. Verify chunks created
    expect(upload.chunks_created).toBeGreaterThan(0);

    // 3. Verify vectors stored
    const query = await searchVectors("workspace-1", randomEmbedding());
    expect(query.length).toBeGreaterThan(0);
  });

  test("Audio upload pipeline", async () => {
    const upload = await uploadDocument("workspace-1", "podcast.mp3");
    expect(upload.success).toBe(true);
    expect(upload.transcription_method).toBeDefined();
    expect(upload.chunks_created).toBeGreaterThan(0);
  });

  test("Archive extraction pipeline", async () => {
    const upload = await uploadDocument("workspace-1", "docs.zip");
    expect(upload.files_extracted).toBeGreaterThan(1);
    expect(upload.processed_files).toBe(upload.files_extracted);
  });
});
```

### 2.2: End-to-End Web Scraping

```javascript
describe("Integration: Web Scrape → Vectors", () => {
  test("scrape webpage and index", async () => {
    const scrape = await scrapeAndIndex("workspace-1", "https://example.com");
    expect(scrape.success).toBe(true);
    expect(scrape.chunks_created).toBeGreaterThan(0);

    // Verify searchable
    const query = await searchVectors("workspace-1", embedText("example"));
    expect(query.some(r => r.metadata.source.includes("example.com"))).toBe(true);
  });

  test("scrape YouTube and index", async () => {
    const scrape = await scrapeAndIndex("workspace-1", "https://youtube.com/watch?v=test", { type: "youtube" });
    expect(scrape.transcript).toBeDefined();
    expect(scrape.chunks_created).toBeGreaterThan(0);
  });

  test("scrape GitHub repo and index", async () => {
    const scrape = await scrapeAndIndex("workspace-1", "https://github.com/user/repo", { type: "github" });
    expect(scrape.files_processed).toBeGreaterThan(10);
    expect(scrape.code_files).toBeGreaterThan(0);
  });
});
```

### 2.3: End-to-End Synthesis

```javascript
describe("Integration: Query → Synthesis", () => {
  beforeAll(async () => {
    // Index test documents
    await uploadDocument("test-ws", "fixtures/research1.pdf");
    await uploadDocument("test-ws", "fixtures/research2.md");
    await uploadDocument("test-ws", "fixtures/research3.docx");
  });

  test("summarize mode", async () => {
    const result = await query("test-ws", {
      query: "What are the key findings?",
      mode: "summarize"
    });
    expect(result.synthesis).toContain("# Summary");
    expect(result.sources.length).toBeGreaterThan(0);
    expect(result.synthesis_time_ms).toBeLessThan(30000);
  });

  test("FAQ mode", async () => {
    const result = await query("test-ws", {
      query: "Generate FAQ",
      mode: "faq"
    });
    expect(result.synthesis).toMatch(/Q:/);
    expect(result.synthesis).toMatch(/A:/);
  });

  test("insights mode", async () => {
    const result = await query("test-ws", {
      query: "Extract key insights",
      mode: "insights"
    });
    expect(result.synthesis).toMatch(/^-/m); // Bullet points
  });

  test("custom prompt mode", async () => {
    const result = await query("test-ws", {
      query: "analyze",
      mode: "custom",
      custom_prompt: "Create a table comparing the approaches"
    });
    expect(result.synthesis).toMatch(/\|/); // Markdown table
  });
});
```

---

## Level 3: Quality Tests (Automated + Manual)

### 3.1: Synthesis Quality Scoring

For each synthesis output, score on 4 dimensions (0-10 each):

| Dimension | Definition | Scoring Criteria |
|-----------|------------|------------------|
| **Factual Accuracy** | Synthesis contains only facts from source docs | 10: Perfect accuracy, 0: Hallucinations |
| **Completeness** | Synthesis covers all relevant information | 10: All key points, 0: Major gaps |
| **Clarity** | Output is well-structured and readable | 10: Professional quality, 0: Confusing |
| **Actionability** | Output provides clear next steps or insights | 10: Immediately useful, 0: Vague platitudes |

**Total Quality Score:** (Factual + Completeness + Clarity + Actionability) / 4

**Pass Threshold:** >8.0/10 average across all test scenarios

### 3.2: Quality Test Scenarios

```javascript
describe("Quality Tests", () => {
  const scoreQuality = (synthesis, sources) => {
    // Manual scoring by human evaluator
    // Returns { factual: 0-10, completeness: 0-10, clarity: 0-10, actionability: 0-10 }
  };

  test("Scenario 1: Research paper synthesis", async () => {
    await uploadDocument("quality-1", "fixtures/eval/paper1.pdf");
    await uploadDocument("quality-1", "fixtures/eval/paper2.pdf");

    const result = await query("quality-1", {
      query: "Compare the two approaches to RAG optimization",
      mode: "briefing"
    });

    const scores = scoreQuality(result.synthesis, result.sources);
    expect(scores.factual).toBeGreaterThan(8);
    expect(scores.completeness).toBeGreaterThan(8);
    expect(scores.clarity).toBeGreaterThan(8);
    expect(scores.actionability).toBeGreaterThan(8);
  });

  test("Scenario 2: Multi-format synthesis", async () => {
    await uploadDocument("quality-2", "fixtures/eval/data.xlsx");
    await uploadDocument("quality-2", "fixtures/eval/report.docx");
    await uploadDocument("quality-2", "fixtures/eval/notes.md");

    const result = await query("quality-2", {
      query: "Analyze the quarterly results",
      mode: "summarize"
    });

    const scores = scoreQuality(result.synthesis, result.sources);
    const avg = (scores.factual + scores.completeness + scores.clarity + scores.actionability) / 4;
    expect(avg).toBeGreaterThan(8.0);
  });

  test("Scenario 3: Audio transcription quality", async () => {
    await uploadDocument("quality-3", "fixtures/eval/podcast.mp3");

    const result = await query("quality-3", {
      query: "What were the main discussion points?",
      mode: "insights"
    });

    const scores = scoreQuality(result.synthesis, result.sources);
    expect(scores.factual).toBeGreaterThan(7); // Lower threshold for audio (transcription errors)
    expect(scores.completeness).toBeGreaterThan(8);
  });

  test("Scenario 4: Code repository analysis", async () => {
    await scrapeAndIndex("quality-4", "https://github.com/user/repo", { type: "github" });

    const result = await query("quality-4", {
      query: "Explain the architecture of this codebase",
      mode: "briefing"
    });

    const scores = scoreQuality(result.synthesis, result.sources);
    expect(scores.factual).toBeGreaterThan(8);
    expect(scores.clarity).toBeGreaterThan(8);
    expect(scores.actionability).toBeGreaterThan(7);
  });

  test("Scenario 5: Web scraping synthesis", async () => {
    await scrapeAndIndex("quality-5", "https://docs.example.com/guide");
    await scrapeAndIndex("quality-5", "https://docs.example.com/api");

    const result = await query("quality-5", {
      query: "Create a getting started guide",
      mode: "custom",
      custom_prompt: "Write a step-by-step tutorial for beginners"
    });

    const scores = scoreQuality(result.synthesis, result.sources);
    expect(scores.completeness).toBeGreaterThan(8);
    expect(scores.actionability).toBeGreaterThan(9); // Tutorial should be highly actionable
  });
});
```

---

## Level 4: Real-World Scenarios (Blind Eval)

### Scenario 1: Polymarket Research (Multi-Format)

**Setup:**
```bash
# Workspace: polymarket-research
# Documents:
- polymarket_whitepaper.pdf
- api_documentation.md
- trading_strategy.docx
- market_data.xlsx
- podcast_interview.mp3 (CTO discussing architecture)
- github:polymarket/contracts (Solidity code)
```

**Query:**
"Analyze Polymarket's architecture and identify arbitrage opportunities. Include technical implementation details and code examples."

**Expected Output:**
- ✅ Technical summary of Polymarket architecture
- ✅ Arbitrage strategies with code snippets
- ✅ References to whitepaper sections, API docs, and contract code
- ✅ Actionable implementation plan

**Evaluation Criteria:**
- Correctly extracts info from all 6 sources
- Synthesizes cross-format (PDF + audio + code)
- Provides working code examples
- Quality score >8.0

---

### Scenario 2: Hyperliquid Trading Strategy (Web + Audio)

**Setup:**
```bash
# Workspace: hyperliquid-strat
# Documents:
- https://hyperliquid.xyz/docs/api
- https://youtube.com/watch?v=hyperliquid-tutorial
- https://blog.hyperliquid.xyz/funding-rates
- internal_notes.txt (Rob's trading observations)
```

**Query:**
"Propose a funding rate arbitrage strategy for Hyperliquid. Include risk analysis and backtesting approach."

**Expected Output:**
- ✅ Strategy description with math
- ✅ Risk factors from blog + video
- ✅ Code for backtesting
- ✅ Incorporates Rob's notes

**Evaluation Criteria:**
- Correctly transcribes and synthesizes YouTube tutorial
- Scrapes blog posts accurately
- Combines multiple web sources
- Quality score >8.0

---

### Scenario 3: Next.js Migration Guide (GitHub + Docs)

**Setup:**
```bash
# Workspace: nextjs-migration
# Documents:
- github:vercel/next.js (focus on app router docs)
- https://nextjs.org/docs/app
- https://github.com/user/legacy-project (existing codebase)
- migration_checklist.md
```

**Query:**
"Create a migration plan from pages router to app router for our project. Identify breaking changes and provide code migration examples."

**Expected Output:**
- ✅ Detailed migration steps
- ✅ Breaking changes specific to the legacy project
- ✅ Before/after code examples
- ✅ Checklist verification

**Evaluation Criteria:**
- Clones and analyzes GitHub repos
- Cross-references official docs
- Provides project-specific guidance
- Quality score >8.5

---

### Scenario 4: Confluence Wiki Synthesis (Enterprise)

**Setup:**
```bash
# Workspace: company-wiki
# Documents:
- https://company.atlassian.net/wiki/spaces/ENG/pages/12345 (architecture)
- https://company.atlassian.net/wiki/spaces/ENG/pages/12346 (deployment)
- https://company.atlassian.net/wiki/spaces/ENG/pages/12347 (troubleshooting)
- incidents.xlsx (past incidents)
```

**Query:**
"Create an on-call runbook combining architecture knowledge, deployment procedures, and past incident resolutions."

**Expected Output:**
- ✅ Structured runbook with sections
- ✅ Links to wiki pages
- ✅ Incident history analysis
- ✅ Troubleshooting flowchart

**Evaluation Criteria:**
- Authenticates and scrapes Confluence
- Preserves wiki hierarchy
- Synthesizes across structured data (xlsx) and unstructured (wiki)
- Quality score >8.0

---

### Scenario 5: Multi-Language Research (OCR + Audio + Code)

**Setup:**
```bash
# Workspace: ml-research
# Documents:
- research_paper.pdf (with equations as images - requires OCR)
- lecture_slides.pptx
- professor_lecture.mp4 (extract audio, transcribe)
- github:paperswithcode/implementation
- handwritten_notes.png (OCR)
```

**Query:**
"Explain the algorithm from the paper, lecture, and code. Include mathematical formulas and implementation details."

**Expected Output:**
- ✅ Algorithm explanation
- ✅ Extracted equations from OCR
- ✅ Lecture insights from audio
- ✅ Code walkthrough from GitHub
- ✅ Synthesizes handwritten notes

**Evaluation Criteria:**
- OCR extracts equations correctly
- Audio transcription captures technical terms
- Code analysis is accurate
- Quality score >7.5 (lower threshold due to OCR/audio difficulty)

---

## Pass/Fail Criteria

### Critical (Must Pass)
- ✅ **100% unit tests pass** (all file formats, connectors, processing)
- ✅ **100% integration tests pass** (end-to-end pipelines work)
- ✅ **All 5 blind scenarios complete** (no crashes, errors handled gracefully)
- ✅ **Average quality score >8.0** across blind scenarios

### Important (Should Pass)
- ✅ **Performance targets met:**
  - PDF processing: <2s for 10 pages
  - Vector search: <100ms (p95)
  - Synthesis: <30s (p95)
- ✅ **File format coverage >95%** (48/50 formats)
- ✅ **Web scraping success rate >90%** (handle bot detection gracefully)
- ✅ **Audio transcription WER <10%** (clear speech)
- ✅ **OCR accuracy >90%** (clean text images)

### Nice to Have
- Audio transcription handles heavy accents
- OCR works on handwritten text
- Code analysis detects complex patterns (design patterns, anti-patterns)

---

## Evaluation Procedure

### Phase 1: Automated Tests (Day 1)
1. Run unit tests: `npm test -- --coverage`
2. Run integration tests: `npm run test:integration`
3. Generate coverage report: `npm run test:coverage`
4. Verify: 100% unit + integration pass

### Phase 2: Quality Scoring (Day 2)
1. Execute 5 quality test scenarios
2. Human evaluator scores each output (4 dimensions x 0-10)
3. Calculate average quality score
4. Verify: Average >8.0

### Phase 3: Blind Evaluation (Day 3)
1. Execute 5 real-world scenarios (Billi runs these)
2. Evaluator only sees input/output (no code visibility)
3. Score each scenario (4 dimensions x 0-10)
4. Verify: All scenarios complete + average >8.0

### Phase 4: Performance Testing (Day 4)
1. Load test: 10 concurrent uploads
2. Stress test: 100k vectors per workspace
3. Latency test: p95 measurements
4. Verify: All performance targets met

---

## Test Data Requirements

### Fixtures Needed
```
fixtures/
├── simple.pdf (3 pages, plain text)
├── metadata.pdf (author, title, date)
├── document.docx (paragraphs, tables)
├── presentation.pptx (10 slides, speaker notes)
├── spreadsheet.xlsx (3 sheets, formulas)
├── structure.md (headings, code blocks, links)
├── data.json (valid JSON)
├── data.csv (100 rows, headers)
├── code/ (50+ language samples)
├── media/
│   ├── clean_text.png (OCR test)
│   ├── clear_speech.wav (transcription test)
│   ├── video.mp4 (audio extraction test)
│   └── handwritten_notes.png (challenging OCR)
├── archive.zip (10 files)
└── eval/ (blind test documents)
```

---

## Success Declaration

Hucki passes this evaluation if:
- ✅ All automated tests pass (100%)
- ✅ Quality score >8.0 average
- ✅ All 5 blind scenarios complete successfully
- ✅ Performance targets met

**Next:** Execute comprehensive build with GSD workflow, create test fixtures, run full eval suite.
