# /test-rag

Comprehensive testing workflow for the AI BotKit RAG (Retrieval-Augmented Generation) engine, covering vector search quality, response accuracy, and performance.

## Context

The RAG engine is the core of AI BotKit's chatbot functionality:
- Converts user queries to embeddings
- Searches Pinecone for relevant documents
- Constructs prompts with context
- Generates streaming responses via LLM

Testing ensures:
- Semantic search returns relevant results
- Context is properly included in prompts
- Responses are accurate and grounded
- Performance meets SLA requirements

## RAG Testing Dimensions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RAG TESTING DIMENSIONS                            │
│                                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │
│  │  RETRIEVAL  │  │   CONTEXT   │  │  GENERATION │  │ PERFORMANCE│ │
│  │   QUALITY   │  │  RELEVANCE  │  │   ACCURACY  │  │    SLA     │ │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤  ├────────────┤ │
│  │ - Precision │  │ - Score     │  │ - Grounded  │  │ - Latency  │ │
│  │ - Recall    │  │   threshold │  │ - Halluc.   │  │ - Tokens   │ │
│  │ - Top-K     │  │ - Context   │  │ - Fallback  │  │ - Stream   │ │
│  │   coverage  │  │   window    │  │   handling  │  │   speed    │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

## Usage

```bash
/test-rag [test-type] [options]
```

### Test Types

| Type | Description |
|------|-------------|
| `all` | Run all tests (default) |
| `retrieval` | Test vector search quality |
| `context` | Test context relevance |
| `generation` | Test response accuracy |
| `performance` | Test latency and throughput |
| `e2e` | End-to-end conversation tests |

### Options

| Option | Description |
|--------|-------------|
| `--chatbot <id>` | Test specific chatbot |
| `--namespace <ns>` | Test specific namespace |
| `--verbose` | Detailed output |
| `--save-report` | Save report to file |

### Examples

```bash
/test-rag                           # Run all tests
/test-rag retrieval                 # Test retrieval only
/test-rag --chatbot 123             # Test specific chatbot
/test-rag performance --verbose     # Detailed performance test
```

## Test Suites

### 1. Retrieval Quality Tests

**Purpose:** Verify semantic search returns relevant documents

**Test Cases:**

| Test | Query | Expected | Pass Criteria |
|------|-------|----------|---------------|
| Exact match | Query using exact doc phrase | Doc returned in top-3 | Score > 0.9 |
| Semantic match | Paraphrased query | Related doc in top-5 | Score > 0.7 |
| No match | Unrelated query | No high-score results | All scores < 0.5 |
| Multi-doc | Query spanning topics | Multiple relevant docs | 3+ docs > 0.7 |

**Test Implementation:**

```typescript
// Test: Semantic search returns relevant documents
async function testRetrieval(chatbotId: number) {
  const testCases = [
    {
      name: 'Exact phrase match',
      query: 'How do I reset my password?', // Exact phrase from docs
      expectedMinScore: 0.9,
      expectedInTop: 3,
    },
    {
      name: 'Semantic similarity',
      query: 'I forgot my login credentials', // Paraphrased
      expectedMinScore: 0.7,
      expectedInTop: 5,
    },
    {
      name: 'No relevant content',
      query: 'What is the weather in Tokyo?', // Unrelated
      expectedMaxScore: 0.5,
      expectedInTop: 0,
    },
  ];

  const results = [];
  for (const testCase of testCases) {
    const searchResults = await pinecone.search(testCase.query, {
      namespace: getNamespace(chatbotId),
      topK: 5,
    });

    const passed = evaluateRetrieval(searchResults, testCase);
    results.push({ ...testCase, passed, results: searchResults });
  }

  return results;
}
```

### 2. Context Relevance Tests

**Purpose:** Verify context window is properly constructed

**Test Cases:**

| Test | Scenario | Pass Criteria |
|------|----------|---------------|
| Context included | Query with docs | Context in prompt |
| Score threshold | Low-score results | Filtered out |
| Context length | Large documents | Properly truncated |
| Deduplication | Similar docs | No duplicates |

**Test Implementation:**

```typescript
async function testContextConstruction(chatbotId: number) {
  const testQuery = 'How do subscriptions work?';

  // Get raw search results
  const searchResults = await pinecone.search(testQuery, {
    namespace: getNamespace(chatbotId),
    topK: 10,
  });

  // Get constructed context
  const context = await ragEngine.buildContext(searchResults, {
    scoreThreshold: 0.7,
    maxTokens: 2000,
  });

  // Verify
  const tests = {
    hasContext: context.length > 0,
    noLowScoreResults: !context.includes(/* low score content */),
    withinTokenLimit: countTokens(context) <= 2000,
    noDuplicates: new Set(context.split('\n\n')).size === context.split('\n\n').length,
  };

  return tests;
}
```

### 3. Generation Accuracy Tests

**Purpose:** Verify responses are grounded in context and accurate

**Test Cases:**

| Test | Scenario | Pass Criteria |
|------|----------|---------------|
| Grounded response | Query with context | Answer from context |
| Hallucination check | Query with partial context | No fabricated info |
| Fallback trigger | No relevant context | Fallback message |
| Tone adherence | Any query | Matches chatbot tone |

**Test Implementation:**

```typescript
async function testGeneration(chatbotId: number) {
  const chatbot = await getChatbot(chatbotId);

  const testCases = [
    {
      name: 'Grounded response',
      query: 'What are your pricing plans?',
      contextContains: 'pricing', // Doc must contain this
      responseShouldContain: ['free', 'basic', 'premium'], // From context
      responseShouldNotContain: ['enterprise'], // Not in context
    },
    {
      name: 'Fallback handling',
      query: 'What is quantum computing?',
      expectFallback: true,
      fallbackMessage: chatbot.messagesTemplate.fallback,
    },
    {
      name: 'Tone adherence',
      query: 'Hello',
      expectedTone: chatbot.messagesTemplate.tone, // 'professional'
    },
  ];

  const results = [];
  for (const testCase of testCases) {
    const response = await ragEngine.generateResponse(testCase.query, chatbotId);
    const passed = evaluateGeneration(response, testCase);
    results.push({ ...testCase, passed, response });
  }

  return results;
}
```

### 4. Performance Tests

**Purpose:** Verify RAG meets latency and throughput requirements

**Metrics:**

| Metric | Target | Critical |
|--------|--------|----------|
| Search latency | < 200ms | > 500ms |
| Context build | < 50ms | > 100ms |
| LLM first token | < 500ms | > 1000ms |
| Total E2E | < 2000ms | > 5000ms |
| Tokens/second | > 20 | < 10 |

**Test Implementation:**

```typescript
async function testPerformance(chatbotId: number) {
  const iterations = 10;
  const metrics = {
    searchLatency: [],
    contextBuild: [],
    firstToken: [],
    totalE2E: [],
    tokensPerSecond: [],
  };

  for (let i = 0; i < iterations; i++) {
    const query = testQueries[i % testQueries.length];

    // Measure search
    const searchStart = performance.now();
    const searchResults = await pinecone.search(query, { namespace, topK: 5 });
    metrics.searchLatency.push(performance.now() - searchStart);

    // Measure context build
    const contextStart = performance.now();
    const context = await ragEngine.buildContext(searchResults);
    metrics.contextBuild.push(performance.now() - contextStart);

    // Measure generation
    const genStart = performance.now();
    let firstTokenTime = 0;
    let tokenCount = 0;

    for await (const token of ragEngine.streamResponse(query, context)) {
      if (firstTokenTime === 0) {
        firstTokenTime = performance.now() - genStart;
      }
      tokenCount++;
    }

    const totalTime = performance.now() - genStart;
    metrics.firstToken.push(firstTokenTime);
    metrics.totalE2E.push(performance.now() - searchStart);
    metrics.tokensPerSecond.push(tokenCount / (totalTime / 1000));
  }

  return {
    searchLatency: { avg: avg(metrics.searchLatency), p95: p95(metrics.searchLatency) },
    contextBuild: { avg: avg(metrics.contextBuild), p95: p95(metrics.contextBuild) },
    firstToken: { avg: avg(metrics.firstToken), p95: p95(metrics.firstToken) },
    totalE2E: { avg: avg(metrics.totalE2E), p95: p95(metrics.totalE2E) },
    tokensPerSecond: { avg: avg(metrics.tokensPerSecond), min: min(metrics.tokensPerSecond) },
  };
}
```

### 5. End-to-End Conversation Tests

**Purpose:** Test complete conversation flows

**Test Scenarios:**

```typescript
const conversationTests = [
  {
    name: 'Single turn Q&A',
    turns: [
      { user: 'What is AI BotKit?', expectContains: ['chatbot', 'AI'] },
    ],
  },
  {
    name: 'Multi-turn with context',
    turns: [
      { user: 'Tell me about pricing', expectContains: ['plan'] },
      { user: 'What about the free tier?', expectContains: ['free', 'messages'] },
      { user: 'How do I upgrade?', expectContains: ['upgrade', 'billing'] },
    ],
  },
  {
    name: 'Edge case handling',
    turns: [
      { user: '', expectFallback: true }, // Empty message
      { user: 'a'.repeat(10000), expectError: true }, // Too long
      { user: '<script>alert(1)</script>', expectSafe: true }, // XSS attempt
    ],
  },
];
```

## Output Format

```markdown
# RAG Engine Test Report

## Summary

| Category | Tests | Passed | Failed | Pass Rate |
|----------|-------|--------|--------|-----------|
| Retrieval | 10 | 9 | 1 | 90% |
| Context | 8 | 8 | 0 | 100% |
| Generation | 12 | 11 | 1 | 92% |
| Performance | 5 | 4 | 1 | 80% |
| E2E | 6 | 6 | 0 | 100% |
| **Total** | **41** | **38** | **3** | **93%** |

## Test Environment

| Property | Value |
|----------|-------|
| Chatbot ID | 123 |
| Namespace | my-bot-user-1 |
| Documents | 45 |
| Pinecone Index | aibotkit-prod |
| LLM Provider | Together AI |

---

## Retrieval Tests

### ✅ Exact phrase match
- **Query:** "How do I reset my password?"
- **Top Result Score:** 0.94
- **Expected:** > 0.9
- **Status:** PASS

### ❌ Semantic similarity - edge case
- **Query:** "credentials recovery process"
- **Top Result Score:** 0.65
- **Expected:** > 0.7
- **Status:** FAIL
- **Recommendation:** Add more training data for password recovery variations

---

## Performance Tests

### Latency Metrics

| Metric | Avg | P95 | Target | Status |
|--------|-----|-----|--------|--------|
| Search | 145ms | 220ms | < 200ms | ⚠️ P95 exceeds |
| Context Build | 32ms | 48ms | < 50ms | ✅ Pass |
| First Token | 380ms | 520ms | < 500ms | ⚠️ P95 exceeds |
| Total E2E | 1.2s | 1.8s | < 2s | ✅ Pass |
| Tokens/sec | 28 | 22 | > 20 | ✅ Pass |

### Performance Chart

```
Latency Distribution (ms)
Search:    [################    ] 145ms avg
Context:   [######              ] 32ms avg
First Tok: [####################] 380ms avg
E2E:       [########################] 1.2s avg
```

---

## Recommendations

### Critical
1. **Retrieval:** Add training data for password recovery variations
2. **Performance:** Consider caching frequent queries

### Improvements
1. Increase score threshold from 0.7 to 0.75 to reduce noise
2. Add query preprocessing for common misspellings

---

## Test Artifacts

- Full logs: `reports/rag-test-20241215.log`
- Failed queries: `reports/rag-test-failures.json`
- Performance data: `reports/rag-performance.csv`
```

## Integration with CI/CD

```yaml
# .github/workflows/rag-tests.yml
name: RAG Tests

on:
  push:
    paths:
      - 'src/lib/ai/**'
      - 'src/app/api/chat/**'

jobs:
  test-rag:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run RAG Tests
        run: |
          pnpm install
          pnpm test:rag
        env:
          PINECONE_API_KEY: ${{ secrets.PINECONE_API_KEY }}
          TOGETHER_API_KEY: ${{ secrets.TOGETHER_API_KEY }}
```

## Related Commands

- `/full-review` - Code review including RAG engine
- `/deploy-saas` - Deployment workflow
- `/next-phase` - Development lifecycle

## Success Criteria

A successful RAG test run:
- ✅ All retrieval tests pass (or documented exceptions)
- ✅ Context properly constructed
- ✅ No hallucinations detected
- ✅ Performance within SLA
- ✅ E2E conversations complete successfully
- ✅ Report generated with recommendations
