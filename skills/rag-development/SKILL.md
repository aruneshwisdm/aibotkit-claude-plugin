---
name: rag-development
description: Best practices for building RAG (Retrieval-Augmented Generation) systems
---

# RAG Development Patterns

Best practices for building production-ready RAG systems like AI BotKit's chatbot engine.

## RAG Architecture Overview

```
User Query
    ↓
┌─────────────────────────────────────────────────────────────┐
│                      RAG PIPELINE                            │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   RETRIEVE   │    │   AUGMENT    │    │   GENERATE   │  │
│  │              │    │              │    │              │  │
│  │ - Embed query│    │ - Build      │    │ - Stream     │  │
│  │ - Search     │    │   context    │    │   response   │  │
│  │ - Rank       │    │ - Construct  │    │ - Track      │  │
│  │ - Filter     │    │   prompt     │    │   tokens     │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
    ↓
Response
```

## Document Processing

### Chunking Strategies

**Optimal chunk size:** 500-1000 tokens

```typescript
interface ChunkingStrategy {
  chunkSize: number;      // Target tokens per chunk
  chunkOverlap: number;   // Overlap between chunks
  separators: string[];   // Split points
}

// Recommended settings
const strategy: ChunkingStrategy = {
  chunkSize: 800,
  chunkOverlap: 100,
  separators: ['\n\n', '\n', '. ', ' '],
};
```

**Chunking by content type:**

| Content Type | Chunk Size | Overlap | Strategy |
|--------------|------------|---------|----------|
| Documentation | 800 | 100 | Paragraph-based |
| FAQ | 200-400 | 50 | Question-answer pairs |
| Long articles | 1000 | 150 | Section-based |
| Code | 500 | 100 | Function-based |

### Metadata Enrichment

Always include metadata with chunks:

```typescript
interface ChunkMetadata {
  source: string;       // Original document
  title: string;        // Document title
  chunk_index: number;  // Position in document
  total_chunks: number; // Total chunks from doc
  content_type: string; // 'text' | 'code' | 'faq'
  created_at: string;   // Indexing timestamp
}

// Example chunk
{
  id: 'doc-123-chunk-5',
  values: [...embeddings],
  metadata: {
    text: 'The actual chunk content...',
    source: 'pricing.md',
    title: 'Pricing Plans',
    chunk_index: 5,
    total_chunks: 12,
    content_type: 'text',
    created_at: '2024-01-15T10:30:00Z',
  }
}
```

## Vector Search Best Practices

### Namespace Isolation

Isolate vectors by chatbot/user:

```typescript
// Namespace pattern: {chatbot-slug}-user-{userId}
const namespace = `${slugify(chatbot.name)}-user-${userId}`;

// Search within namespace
const results = await pinecone
  .index(indexName)
  .namespace(namespace)
  .query({
    vector: queryEmbedding,
    topK: 5,
    includeMetadata: true,
  });
```

### Score Thresholding

Filter out low-relevance results:

```typescript
const SCORE_THRESHOLD = 0.7;

const relevantResults = searchResults.matches
  .filter(match => match.score >= SCORE_THRESHOLD)
  .slice(0, 5);

// Handle no results
if (relevantResults.length === 0) {
  return useFallbackResponse();
}
```

### Reranking for Quality

For higher quality, rerank with a cross-encoder:

```typescript
// Simple reranking by query similarity
async function rerankResults(
  query: string,
  results: SearchResult[]
): Promise<SearchResult[]> {
  // Score based on keyword overlap + semantic score
  const reranked = results.map(result => ({
    ...result,
    rerankedScore: calculateRelevance(query, result.metadata.text, result.score),
  }));

  return reranked.sort((a, b) => b.rerankedScore - a.rerankedScore);
}
```

## Prompt Engineering

### System Prompt Template

```typescript
const systemPrompt = `You are ${chatbot.personality}.
Your communication style is ${chatbot.tone}.

CONTEXT (use this information to answer questions):
---
${context}
---

INSTRUCTIONS:
1. Answer questions based ONLY on the provided context
2. If the context doesn't contain the answer, say: "${chatbot.fallbackMessage}"
3. Be concise and helpful
4. Don't make up information not in the context
5. If asked about topics outside your knowledge, politely redirect

CURRENT DATE: ${new Date().toISOString().split('T')[0]}`;
```

### Context Window Management

Balance context size with response quality:

```typescript
const MAX_CONTEXT_TOKENS = 3000;
const MAX_HISTORY_MESSAGES = 10;

function buildPrompt(
  query: string,
  context: string,
  history: Message[]
): Message[] {
  // Truncate context if needed
  const truncatedContext = truncateToTokens(context, MAX_CONTEXT_TOKENS);

  // Keep recent history
  const recentHistory = history.slice(-MAX_HISTORY_MESSAGES);

  return [
    { role: 'system', content: systemPrompt.replace('${context}', truncatedContext) },
    ...recentHistory,
    { role: 'user', content: query },
  ];
}
```

### Token Budget Allocation

```
Total Context Window: 8192 tokens (typical)
├── System Prompt:     500 tokens (fixed)
├── Retrieved Context: 3000 tokens (variable)
├── Conversation History: 2000 tokens (sliding)
├── User Query:        200 tokens (typical)
└── Response Buffer:   2492 tokens (for generation)
```

## Streaming Responses

### Server-Sent Events (SSE)

```typescript
export async function POST(request: NextRequest) {
  const { chatbotId, message, sessionId } = await request.json();

  // Validate and get context
  const context = await retrieveContext(chatbotId, message);
  const messages = await buildPrompt(message, context, sessionId);

  // Create streaming response
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();

      try {
        for await (const chunk of llmClient.stream(messages)) {
          const data = JSON.stringify({ content: chunk });
          controller.enqueue(encoder.encode(`data: ${data}\n\n`));
        }
        controller.enqueue(encoder.encode('data: [DONE]\n\n'));
      } catch (error) {
        const errorData = JSON.stringify({ error: 'Generation failed' });
        controller.enqueue(encoder.encode(`data: ${errorData}\n\n`));
      } finally {
        controller.close();
      }
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

### Client-Side Handling

```typescript
async function streamChat(message: string): AsyncGenerator<string> {
  const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ chatbotId, message, sessionId }),
  });

  const reader = response.body?.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') return;

        const parsed = JSON.parse(data);
        if (parsed.content) yield parsed.content;
        if (parsed.error) throw new Error(parsed.error);
      }
    }
  }
}
```

## Error Handling

### Graceful Degradation

```typescript
async function generateResponse(query: string, chatbotId: number) {
  try {
    // Try primary flow
    const context = await retrieveContext(chatbotId, query);
    return await generateWithContext(query, context);
  } catch (error) {
    if (error.code === 'PINECONE_UNAVAILABLE') {
      // Fallback: generate without context
      console.warn('Pinecone unavailable, using fallback');
      return await generateWithoutContext(query);
    }

    if (error.code === 'LLM_RATE_LIMITED') {
      // Retry with backoff
      await sleep(1000);
      return await generateResponse(query, chatbotId);
    }

    if (error.code === 'CONTEXT_TOO_LONG') {
      // Truncate and retry
      const shorterContext = truncate(context, 0.5);
      return await generateWithContext(query, shorterContext);
    }

    // Return fallback message
    const chatbot = await getChatbot(chatbotId);
    return chatbot.messagesTemplate.fallback;
  }
}
```

## Performance Optimization

### Caching Strategies

```typescript
// Cache embeddings for repeated queries
const embeddingCache = new Map<string, number[]>();

async function getEmbedding(text: string): Promise<number[]> {
  const cacheKey = hashString(text);

  if (embeddingCache.has(cacheKey)) {
    return embeddingCache.get(cacheKey)!;
  }

  const embedding = await embeddingClient.embed(text);
  embeddingCache.set(cacheKey, embedding);

  return embedding;
}

// Cache frequent search results
const searchCache = new LRUCache<string, SearchResult[]>({
  max: 1000,
  ttl: 1000 * 60 * 5, // 5 minutes
});
```

### Batch Operations

```typescript
// Batch document indexing
async function indexDocuments(documents: Document[]) {
  const BATCH_SIZE = 100;

  for (let i = 0; i < documents.length; i += BATCH_SIZE) {
    const batch = documents.slice(i, i + BATCH_SIZE);

    const vectors = await Promise.all(
      batch.map(async (doc) => ({
        id: doc.id,
        values: await getEmbedding(doc.content),
        metadata: { text: doc.content, ...doc.metadata },
      }))
    );

    await pinecone.index(indexName).namespace(namespace).upsert(vectors);
  }
}
```

## Monitoring and Analytics

### Key Metrics

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Search latency | <200ms | >500ms |
| Generation latency | <2s | >5s |
| Relevance score (avg) | >0.75 | <0.6 |
| Fallback rate | <10% | >25% |
| Error rate | <1% | >5% |

### Logging for Analysis

```typescript
interface RAGAnalytics {
  queryId: string;
  chatbotId: number;
  query: string;
  searchLatencyMs: number;
  topResultScore: number;
  contextTokens: number;
  generationLatencyMs: number;
  responseTokens: number;
  usedFallback: boolean;
  timestamp: Date;
}

async function logRAGMetrics(metrics: RAGAnalytics) {
  await analytics.track('rag_query', metrics);
}
```

## Quality Evaluation

### Evaluation Metrics

1. **Retrieval Quality**
   - Precision@K: Relevant docs in top K
   - Recall: Coverage of relevant docs
   - MRR: Mean Reciprocal Rank

2. **Generation Quality**
   - Groundedness: Response based on context
   - Relevance: Answer addresses question
   - Fluency: Natural language quality

### A/B Testing RAG Changes

```typescript
// Feature flag for RAG experiments
const ragConfig = await getFeatureFlags(chatbotId);

if (ragConfig.useReranking) {
  results = await rerankResults(query, results);
}

if (ragConfig.scoreThreshold) {
  results = results.filter(r => r.score >= ragConfig.scoreThreshold);
}
```
