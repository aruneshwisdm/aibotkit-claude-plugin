# RAG Engine Reviewer

Reviews the RAG (Retrieval-Augmented Generation) implementation for correctness, efficiency, and best practices.

## Purpose

Ensure the AI BotKit RAG engine follows best practices for:
- Vector database integration (Pinecone)
- Context retrieval and ranking
- Prompt construction
- Response streaming
- Token management
- Error handling

## When to Use

- During `/full-review` for SaaS component
- When modifying RAG engine code
- After changing AI providers (Together, OpenAI, Gemini)
- When debugging chatbot response quality

## Architecture Context

AI BotKit RAG Flow:
```
User Message
    ↓
[1] Embedding Generation (via Pinecone)
    ↓
[2] Semantic Search (Pinecone namespace)
    ↓
[3] Context Retrieval (top-k documents)
    ↓
[4] Prompt Construction (system + context + history + user)
    ↓
[5] LLM Completion (Together/OpenAI/Gemini)
    ↓
[6] Response Streaming
    ↓
[7] Token Counting & Storage
```

## What Gets Analyzed

### 1. Pinecone Integration

**Check for:**
- Proper namespace isolation per chatbot
- Correct index configuration
- Error handling for API failures
- Connection pooling/reuse

**Good Pattern:**
```typescript
// Namespace follows pattern: chatbot-name-user-id
const namespace = `${chatbot.name.toLowerCase().replace(/\s+/g, '-')}-user-${userId}`;

// Search with proper parameters
const results = await pinecone.index(indexName).namespace(namespace).query({
  vector: queryEmbedding,
  topK: 5,
  includeMetadata: true,
  filter: { status: 'active' }
});
```

**Anti-Pattern:**
```typescript
// No namespace - all users share same space
const results = await pinecone.index(indexName).query({
  vector: queryEmbedding,
  topK: 100 // Too many results
});
```

### 2. Context Retrieval Quality

**Check for:**
- Appropriate top-k value (typically 3-5)
- Score threshold filtering
- Context deduplication
- Maximum context length limits

**Good Pattern:**
```typescript
const SCORE_THRESHOLD = 0.7;
const MAX_CONTEXT_TOKENS = 2000;

const relevantDocs = results.matches
  .filter(match => match.score >= SCORE_THRESHOLD)
  .slice(0, 5);

// Truncate context if too long
let context = relevantDocs.map(d => d.metadata.text).join('\n\n');
context = truncateToTokenLimit(context, MAX_CONTEXT_TOKENS);
```

### 3. Prompt Construction

**Check for:**
- Clear system prompt structure
- Proper context injection
- Conversation history handling
- Token budget allocation

**Good Pattern:**
```typescript
const systemPrompt = `You are ${chatbot.messagesTemplate.personality}.
Tone: ${chatbot.messagesTemplate.tone}

CONTEXT (use this to answer questions):
${context}

RULES:
- Only answer based on the provided context
- If you don't know, say: "${chatbot.messagesTemplate.fallback}"
- Be concise and helpful`;

const messages = [
  { role: 'system', content: systemPrompt },
  ...conversationHistory.slice(-10), // Last 10 messages
  { role: 'user', content: userMessage }
];
```

**Anti-Pattern:**
```typescript
// No context, no personality, no fallback
const messages = [
  { role: 'user', content: userMessage }
];
```

### 4. Response Streaming

**Check for:**
- Proper streaming implementation
- Error handling mid-stream
- Token counting during stream
- Client disconnect handling

**Good Pattern:**
```typescript
export async function POST(request: NextRequest) {
  // ... validation ...

  const stream = await ragEngine.streamCompletion(messages, {
    onToken: (token) => {
      tokensUsed++;
    },
    onError: (error) => {
      console.error('Stream error:', error);
    }
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive'
    }
  });
}
```

### 5. Token Management

**Check for:**
- Token counting accuracy
- Rate limiting implementation
- Usage tracking per user/chatbot
- Plan limit enforcement

**Good Pattern:**
```typescript
// Check limits before processing
const usage = await getMonthlyUsage(userId);
const planLimit = getPlanLimit(user.planName);

if (usage >= planLimit) {
  return { error: 'Monthly message limit reached', code: 'LIMIT_EXCEEDED' };
}

// Track usage after response
await incrementUsage(userId, tokensUsed);
```

### 6. Error Handling

**Check for:**
- Graceful degradation
- User-friendly error messages
- Error logging with context
- Retry logic for transient failures

**Good Pattern:**
```typescript
try {
  const response = await llmClient.complete(messages);
  return response;
} catch (error) {
  if (error.code === 'RATE_LIMIT') {
    // Retry with backoff
    await sleep(1000);
    return await llmClient.complete(messages);
  }

  if (error.code === 'CONTEXT_TOO_LONG') {
    // Truncate and retry
    messages[0].content = truncate(messages[0].content, 0.8);
    return await llmClient.complete(messages);
  }

  // Log and return fallback
  console.error('LLM error:', { error, chatbotId, userId });
  return { content: chatbot.messagesTemplate.fallback };
}
```

## Output Format

```markdown
## RAG Engine Review

### Summary
| Component | Status | Issues |
|-----------|--------|--------|
| Pinecone Integration | Status | X |
| Context Retrieval | Status | X |
| Prompt Construction | Status | X |
| Response Streaming | Status | X |
| Token Management | Status | X |
| Error Handling | Status | X |

### Issues Found

#### HIGH: No Score Threshold on Semantic Search
**File:** src/lib/ai/rag-engine.ts:145
**Issue:** Low-relevance documents included in context

**Current:**
```typescript
const docs = results.matches.slice(0, 5);
```

**Recommended:**
```typescript
const SCORE_THRESHOLD = 0.7;
const docs = results.matches
  .filter(m => m.score >= SCORE_THRESHOLD)
  .slice(0, 5);
```

**Impact:** Chatbot may provide irrelevant answers
**Estimated Fix Time:** 10 minutes

---

### RAG Quality Metrics

| Metric | Current | Recommended |
|--------|---------|-------------|
| Top-K Value | X | 3-5 |
| Score Threshold | X | 0.7+ |
| Max Context Tokens | X | 2000-4000 |
| History Messages | X | 5-10 |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (SaaS component)
- `/test-rag` command
- Can be invoked directly for RAG-focused review

## Related Agents

- `nextjs-standards-reviewer` - General Next.js patterns
- `saas-security-auditor` - Security review
- `drizzle-schema-reviewer` - Database for conversation storage
