# Hucki - Evaluation Specification

**Product:** Hucki v1.0
**Date:** 2026-03-27
**Owner:** Claudi
**Purpose:** Define success criteria and test methodology

---

## Evaluation Philosophy

**Hucki must be evaluated on:**
1. **Correctness:** Does it work as specified?
2. **Quality:** Does it synthesize research well?
3. **Performance:** Is it fast enough for automation?
4. **Reliability:** Does it fail gracefully?

**Evaluation is blind:** Claudi (builder) doesn't see eval specs until after implementation.

---

## Evaluation Levels

### Level 1: Unit Tests (Automated)
**Purpose:** Verify individual components work
**Run by:** Automated test suite
**Pass criteria:** 100% of unit tests pass

### Level 2: Integration Tests (Automated)
**Purpose:** Verify API endpoints work end-to-end
**Run by:** Automated test suite
**Pass criteria:** 100% of integration tests pass

### Level 3: Quality Eval (Semi-Automated)
**Purpose:** Evaluate synthesis quality vs baseline
**Run by:** Billi (has access to ground truth)
**Pass criteria:** >80% quality score vs human baseline

### Level 4: Real-World Eval (Manual)
**Purpose:** Test with actual IRCA workflow
**Run by:** Rob + Billi
**Pass criteria:** Successfully completes 3/3 real research tasks

---

## Level 1: Unit Test Spec

### Test Suite: Workspace Manager

```javascript
describe("WorkspaceManager", () => {
  test("creates workspace with valid ID", async () => {
    const ws = await WorkspaceManager.create("Test Project");
    expect(ws.id).toMatch(/^[a-z0-9-]+$/);
    expect(ws.name).toBe("Test Project");
    expect(ws.doc_count).toBe(0);
  });

  test("lists all workspaces", async () => {
    await WorkspaceManager.create("WS1");
    await WorkspaceManager.create("WS2");
    const list = await WorkspaceManager.list();
    expect(list.length).toBeGreaterThanOrEqual(2);
  });

  test("deletes workspace and vectors", async () => {
    const ws = await WorkspaceManager.create("Temp");
    await WorkspaceManager.delete(ws.id);
    const exists = await WorkspaceManager.get(ws.id);
    expect(exists).toBeNull();
  });

  test("rejects invalid workspace names", async () => {
    await expect(
      WorkspaceManager.create("")
    ).rejects.toThrow();
  });
});
```

### Test Suite: Document Processor

```javascript
describe("DocumentProcessor", () => {
  test("chunks text into <1000 char segments", () => {
    const text = "A".repeat(2500);
    const chunks = DocumentProcessor.chunkText(text);
    expect(chunks.length).toBe(3);
    chunks.forEach(chunk => {
      expect(chunk.length).toBeLessThanOrEqual(1000);
    });
  });

  test("preserves sentence boundaries", () => {
    const text = "Sentence one. Sentence two. Sentence three.";
    const chunks = DocumentProcessor.chunkText(text, {maxChars: 20});
    // Should not split mid-sentence
    chunks.forEach(chunk => {
      expect(chunk.trim()).toMatch(/^[A-Z].*[.!?]$/);
    });
  });

  test("extracts text from markdown", async () => {
    const md = "# Title\n\nSome **bold** text.";
    const text = await DocumentProcessor.extractText(md, "md");
    expect(text).toContain("Title");
    expect(text).toContain("bold");
  });

  test("handles empty files gracefully", async () => {
    const result = await DocumentProcessor.process("");
    expect(result.chunks).toEqual([]);
  });
});
```

### Test Suite: Embedder

```javascript
describe("Embedder", () => {
  test("generates 384-dim embedding", async () => {
    const embedding = await Embedder.embed("test text");
    expect(embedding.length).toBe(384);
    expect(typeof embedding[0]).toBe("number");
  });

  test("similar texts have similar embeddings", async () => {
    const e1 = await Embedder.embed("machine learning");
    const e2 = await Embedder.embed("deep learning");
    const e3 = await Embedder.embed("banana recipe");

    const sim12 = cosineSimilarity(e1, e2);
    const sim13 = cosineSimilarity(e1, e3);

    expect(sim12).toBeGreaterThan(sim13);
  });

  test("handles non-English text", async () => {
    const embedding = await Embedder.embed("你好世界");
    expect(embedding.length).toBe(384);
  });
});
```

### Test Suite: Convex Vector Store

```javascript
describe("ConvexVectorStore", () => {
  test("inserts vector with workspace isolation", async () => {
    const vector = {
      workspace_id: "test-ws",
      chunk_text: "Sample text",
      embedding: Array(384).fill(0.1),
      source_file: "test.md"
    };

    const id = await ConvexVectorStore.insert(vector);
    expect(id).toBeDefined();
  });

  test("searches within workspace only", async () => {
    // Insert into two workspaces
    await ConvexVectorStore.insert({
      workspace_id: "ws1",
      chunk_text: "Apple",
      embedding: await Embedder.embed("Apple")
    });

    await ConvexVectorStore.insert({
      workspace_id: "ws2",
      chunk_text: "Apple",
      embedding: await Embedder.embed("Apple")
    });

    // Search ws1
    const results = await ConvexVectorStore.search({
      workspace_id: "ws1",
      query_embedding: await Embedder.embed("Apple"),
      limit: 10
    });

    // Should only return ws1 results
    expect(results.every(r => r.workspace_id === "ws1")).toBe(true);
  });

  test("returns top N results", async () => {
    const results = await ConvexVectorStore.search({
      workspace_id: "test",
      query_embedding: Array(384).fill(0.5),
      limit: 5
    });

    expect(results.length).toBeLessThanOrEqual(5);
  });
});
```

### Test Suite: Claude Code Synthesizer

```javascript
describe("ClaudeCodeSynthesizer", () => {
  test("generates context.md file", async () => {
    const chunks = [
      {text: "Finding 1: X is important", source: "doc1.md"},
      {text: "Finding 2: Y correlates with Z", source: "doc2.md"}
    ];

    const result = await ClaudeCodeSynthesizer.synthesize({
      workspace_id: "test",
      chunks: chunks,
      output_path: "/tmp/test-context.md"
    });

    expect(fs.existsSync("/tmp/test-context.md")).toBe(true);
    const content = fs.readFileSync("/tmp/test-context.md", "utf8");
    expect(content).toContain("Executive Summary");
    expect(content).toContain("Key Findings");
  });

  test("handles Claude Code CLI unavailable", async () => {
    // Mock CLI failure
    jest.spyOn(child_process, "exec").mockImplementation((cmd, cb) => {
      cb(new Error("claude: command not found"));
    });

    await expect(
      ClaudeCodeSynthesizer.synthesize({chunks: []})
    ).rejects.toThrow("Claude Code CLI not available");
  });

  test("includes source citations", async () => {
    const chunks = [
      {text: "Data point", source: "source.pdf"}
    ];

    const result = await ClaudeCodeSynthesizer.synthesize({
      workspace_id: "test",
      chunks
    });

    const content = fs.readFileSync(result.file_path, "utf8");
    expect(content).toContain("source.pdf");
  });
});
```

**Pass Criteria:** 25/25 unit tests pass

---

## Level 2: Integration Test Spec

### Test Scenario: Full Workspace Lifecycle

```javascript
describe("Integration: Workspace Lifecycle", () => {
  let workspaceId;

  test("E2E: Create → Index → Query → Synthesize → Delete", async () => {
    // 1. Create workspace
    const createRes = await request(app)
      .post("/workspace/create")
      .send({name: "Integration Test"})
      .expect(200);

    workspaceId = createRes.body.id;
    expect(workspaceId).toBeDefined();

    // 2. Prepare test documents
    const testDir = "/tmp/hucki-test-docs";
    fs.mkdirSync(testDir, {recursive: true});
    fs.writeFileSync(`${testDir}/doc1.md`, "# Research Finding\n\nMean reversion works.");
    fs.writeFileSync(`${testDir}/doc2.md`, "# Analysis\n\nBacktest shows 15% returns.");

    // 3. Index documents
    const indexRes = await request(app)
      .post(`/workspace/${workspaceId}/index`)
      .send({directory: testDir})
      .expect(200);

    expect(indexRes.body.chunks_indexed).toBeGreaterThan(0);

    // 4. Query
    const queryRes = await request(app)
      .post(`/workspace/${workspaceId}/query`)
      .send({
        question: "What are the returns?",
        mode: "retrieve"
      })
      .expect(200);

    expect(queryRes.body.chunks.length).toBeGreaterThan(0);
    expect(queryRes.body.chunks[0].text).toContain("15%");

    // 5. Synthesize
    const synthRes = await request(app)
      .post(`/workspace/${workspaceId}/synthesize`)
      .expect(200);

    expect(fs.existsSync(synthRes.body.context_file)).toBe(true);

    // 6. Delete
    await request(app)
      .delete(`/workspace/${workspaceId}`)
      .expect(200);

    // Verify deleted
    await request(app)
      .get(`/workspace/${workspaceId}/stats`)
      .expect(404);
  });
});
```

### Test Scenario: Project Isolation

```javascript
describe("Integration: Workspace Isolation", () => {
  test("workspaces don't leak data", async () => {
    // Create two workspaces
    const ws1 = await createWorkspace("Project A");
    const ws2 = await createWorkspace("Project B");

    // Index different docs
    await indexDocument(ws1.id, "Document about apples");
    await indexDocument(ws2.id, "Document about oranges");

    // Query ws1 for oranges (shouldn't find)
    const result = await queryWorkspace(ws1.id, "oranges");
    expect(result.chunks.length).toBe(0);

    // Query ws2 for oranges (should find)
    const result2 = await queryWorkspace(ws2.id, "oranges");
    expect(result2.chunks.length).toBeGreaterThan(0);
  });
});
```

### Test Scenario: Error Handling

```javascript
describe("Integration: Error Handling", () => {
  test("handles missing directory gracefully", async () => {
    const ws = await createWorkspace("Test");

    const res = await request(app)
      .post(`/workspace/${ws.id}/index`)
      .send({directory: "/nonexistent/path"})
      .expect(400);

    expect(res.body.error).toContain("Directory not found");
  });

  test("handles Convex downtime", async () => {
    // Mock Convex client failure
    jest.spyOn(ConvexClient.prototype, "query").mockRejectedValue(
      new Error("Network error")
    );

    const res = await request(app)
      .get("/health")
      .expect(503);

    expect(res.body.convex).toBe("unavailable");
  });

  test("handles malformed PDF gracefully", async () => {
    const ws = await createWorkspace("Test");
    const malformedPdf = "/tmp/broken.pdf";
    fs.writeFileSync(malformedPdf, "not a real PDF");

    const res = await request(app)
      .post(`/workspace/${ws.id}/index`)
      .send({file_path: malformedPdf})
      .expect(200);

    // Should skip, not crash
    expect(res.body.chunks_indexed).toBe(0);
    expect(res.body.errors).toContain("broken.pdf");
  });
});
```

**Pass Criteria:** 10/10 integration tests pass

---

## Level 3: Quality Evaluation (Blind)

### Evaluation Dataset

**5 research scenarios with ground truth:**

#### Scenario 1: Polymarket Mean Reversion
**Input:** 3 research documents about mean reversion strategies
**Ground Truth Answer:** "Mean reversion strategy bets on price returning to average. Backtest shows 15% annual returns with 0.6 Sharpe ratio. Best applied to high-volume markets."
**Eval Question:** "Summarize the mean reversion strategy and its performance"

#### Scenario 2: AI Agent Architecture
**Input:** 4 technical papers on AI agents
**Ground Truth Answer:** "AI agents use LLMs + tools + memory. Key patterns: ReAct, Chain-of-Thought, and Tool Use. Performance depends on prompt engineering and tool selection."
**Eval Question:** "What are the key architectural patterns for AI agents?"

#### Scenario 3: Rust vs Go Performance
**Input:** 2 benchmark studies
**Ground Truth Answer:** "Rust is 20-30% faster than Go in CPU-bound tasks. Go excels in concurrent I/O. Memory usage similar. Rust has steeper learning curve."
**Eval Question:** "Compare Rust and Go performance"

#### Scenario 4: Convex vs Firebase
**Input:** 5 comparison articles
**Ground Truth Answer:** "Convex provides reactive queries, TypeScript-first, better DX. Firebase has larger ecosystem, more mature, better for mobile. Convex cheaper at scale."
**Eval Question:** "What are the tradeoffs between Convex and Firebase?"

#### Scenario 5: Neobrutalism Design
**Input:** 3 design articles
**Ground Truth Answer:** "Neobrutalism: bold typography, stark contrast, raw geometric shapes, minimal color. Rejects polish for honest, functional design. Influenced by Swiss design and 90s web."
**Eval Question:** "Describe neobrutalist design principles"

### Evaluation Metrics

**For each scenario:**

1. **Factual Accuracy (0-10)**
   - 10: All facts correct
   - 5: Some facts correct, some missing
   - 0: Hallucinated or wrong facts

2. **Completeness (0-10)**
   - 10: Covers all key points from ground truth
   - 5: Covers half the key points
   - 0: Misses all key points

3. **Clarity (0-10)**
   - 10: Crystal clear, well-organized
   - 5: Understandable but messy
   - 0: Confusing or incoherent

4. **Actionability (0-10)**
   - 10: Provides specific next steps
   - 5: Vague suggestions
   - 0: No actionable insights

**Total Score per Scenario:** 40 points
**Overall Quality Score:** Average across 5 scenarios / 40 * 100 = X%

**Pass Criteria:** >80% quality score

### Evaluation Process

1. **Blind Setup:** Billi prepares dataset, Claudi doesn't see it
2. **Execution:** Claudi indexes research and generates context for each scenario
3. **Scoring:** Billi scores each output vs ground truth
4. **Report:** Billi provides scores + feedback
5. **Iteration:** If <80%, Claudi fixes and re-runs

---

## Level 4: Real-World Evaluation

### Test 1: Polymarket Strategy Research

**Task:** Research Polymarket arbitrage opportunities
**n8n Workflow:**
1. Plane task created: "Research Polymarket arb"
2. n8n searches web for "polymarket arbitrage 2026"
3. n8n saves results to `/root/Research/polymarket-arb/`
4. Hucki indexes and synthesizes
5. Billi reads `context.md` and commissions Claudi to build bot

**Success Criteria:**
- [ ] Workspace created automatically
- [ ] Documents indexed without errors
- [ ] Context.md generated within 30s
- [ ] Context contains actionable strategy
- [ ] Billi can commission Claudi from context

### Test 2: Technical Documentation Synthesis

**Task:** Synthesize Next.js 15 documentation
**n8n Workflow:**
1. Plane task: "Summarize Next.js 15 changes"
2. n8n fetches official docs
3. Saves to `/root/Research/nextjs-15/`
4. Hucki processes
5. Rob reads context to understand changes

**Success Criteria:**
- [ ] Handles large doc set (50+ pages)
- [ ] Identifies breaking changes
- [ ] Provides migration steps
- [ ] Completes in <60s

### Test 3: Multi-Source Integration

**Task:** Research a competitor product
**n8n Workflow:**
1. Plane task: "Research Linear alternatives"
2. n8n gathers: web articles, GitHub issues, Reddit threads
3. Saves to `/root/Research/linear-alternatives/`
4. Hucki synthesizes
5. Rob uses context to decide on Plane

**Success Criteria:**
- [ ] Handles mixed source types
- [ ] Identifies patterns across sources
- [ ] Provides comparison table in context
- [ ] No data leakage from other workspaces

**Pass Criteria:** 3/3 real-world tests succeed

---

## Performance Benchmarks

### Speed Targets

| Operation | Target | Max Acceptable |
|-----------|--------|----------------|
| Create workspace | <100ms | 500ms |
| Index 1 document (1000 words) | <2s | 5s |
| Index 10 documents | <15s | 30s |
| Vector search (top 10) | <200ms | 1s |
| Synthesize context (10 chunks) | <20s | 60s |
| Full workflow (index + synthesize) | <30s | 90s |

### Load Targets

| Metric | Target |
|--------|--------|
| Concurrent workspaces | 10 |
| Max vectors per workspace | 10,000 |
| Max total vectors | 100,000 |
| API requests/min | 60 |

---

## Reliability Tests

### Test: Convex Failure Recovery

```javascript
test("gracefully degrades when Convex unavailable", async () => {
  // Simulate Convex downtime
  mockConvexDown();

  // Should return error, not crash
  const res = await request(app)
    .post("/workspace/test/query")
    .send({question: "test"})
    .expect(503);

  expect(res.body.error).toContain("Vector search unavailable");
});
```

### Test: Claude Code Unavailable

```javascript
test("fallback when Claude Code not found", async () => {
  // Mock missing claude CLI
  process.env.PATH = "/nonexistent";

  const res = await request(app)
    .post("/workspace/test/synthesize")
    .expect(500);

  expect(res.body.error).toContain("Claude Code CLI not available");
  expect(res.body.suggestion).toContain("install claude");
});
```

### Test: Partial Indexing Failure

```javascript
test("continues indexing after single file failure", async () => {
  // Mix good and bad files
  const dir = "/tmp/mixed";
  fs.writeFileSync(`${dir}/good.md`, "Valid content");
  fs.writeFileSync(`${dir}/bad.pdf`, "Corrupted");
  fs.writeFileSync(`${dir}/good2.md`, "More content");

  const res = await indexDirectory(dir);

  expect(res.chunks_indexed).toBe(2); // 2 good files
  expect(res.errors.length).toBe(1); // 1 bad file
  expect(res.errors[0]).toContain("bad.pdf");
});
```

---

## Security Evaluation

### Test: API Authentication

```javascript
test("requires bearer token for API access", async () => {
  await request(app)
    .post("/workspace/create")
    .expect(401);

  await request(app)
    .post("/workspace/create")
    .set("Authorization", "Bearer valid-token")
    .expect(200);
});
```

### Test: Workspace Isolation (Security)

```javascript
test("cannot query other user's workspace", async () => {
  const ws1 = await createWorkspace("User 1 WS");
  const ws2 = await createWorkspace("User 2 WS");

  // Try to query ws1 with ws2's context
  // (In multi-tenant future, this should fail)
  // For now, single-tenant, so this is informational
});
```

---

## Evaluation Report Template

```markdown
# Hucki Evaluation Report

**Date:** YYYY-MM-DD
**Evaluator:** Billi
**Version:** v1.0.0

## Summary
- Overall Pass: YES / NO
- Quality Score: X%
- Performance: PASS / FAIL
- Reliability: PASS / FAIL

## Level 1: Unit Tests
- Tests Run: X
- Tests Passed: X
- Coverage: X%
- Status: PASS / FAIL

## Level 2: Integration Tests
- Tests Run: X
- Tests Passed: X
- Status: PASS / FAIL

## Level 3: Quality Evaluation
- Scenario 1: X/40 (Y%)
- Scenario 2: X/40 (Y%)
- Scenario 3: X/40 (Y%)
- Scenario 4: X/40 (Y%)
- Scenario 5: X/40 (Y%)
- **Average: X%** (Target: >80%)
- Status: PASS / FAIL

## Level 4: Real-World Tests
- Test 1 (Polymarket): PASS / FAIL
- Test 2 (Docs): PASS / FAIL
- Test 3 (Multi-source): PASS / FAIL
- Status: PASS / FAIL

## Performance Benchmarks
[Table of actual vs target times]

## Issues Found
1. [Issue description]
2. [Issue description]

## Recommendations
1. [Recommendation]
2. [Recommendation]

## Final Verdict
SHIP / ITERATE
```

---

## Success Criteria Summary

**Hucki v1.0 is READY TO SHIP when:**
- [x] 100% unit tests pass (25/25)
- [x] 100% integration tests pass (10/10)
- [x] >80% quality score (blind eval)
- [x] 3/3 real-world tests pass
- [x] All performance targets met
- [x] Reliability tests pass
- [x] Security tests pass
- [x] Documentation complete

**If any criteria fail:** Iterate and re-evaluate.

---

**Next Document: GSD Build Plan**
