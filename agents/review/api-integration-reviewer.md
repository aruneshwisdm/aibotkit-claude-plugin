# API Integration Reviewer

Reviews the integration between the AI BotKit SaaS application and WordPress plugin for consistency, security, and reliability.

## Purpose

Ensure seamless and secure communication between:
- SaaS REST API endpoints
- WordPress plugin API client
- Authentication/token handling
- Error handling and recovery

## When to Use

- During `/full-review` command
- When modifying API endpoints
- When changing authentication flow
- After API version updates

## Architecture Context

AI BotKit Integration Flow:
```
WordPress Plugin                        SaaS Application
      |                                       |
      |  1. Connect Site (token exchange)     |
      | ------------------------------------> |
      |                                       |
      |  2. Fetch Chatbots                    |
      | ------------------------------------> |
      |      (Authorization: Bearer token)    |
      |                                       |
      |  3. Render Widget                     |
      | <------------------------------------ |
      |      (chatbot config JSON)            |
      |                                       |
      |  4. User sends message                |
      | ------------------------------------> |
      |      (chatbotId, sessionId, message)  |
      |                                       |
      |  5. Stream response                   |
      | <------------------------------------ |
      |      (SSE stream)                     |
```

## What Gets Analyzed

### 1. API Endpoint Consistency

**Check for:**
- Matching endpoint paths
- Consistent request/response formats
- Version compatibility
- Breaking changes

**Good Pattern:**
```typescript
// SaaS: src/app/api/wordpress/chatbots/route.ts
export async function GET(request: NextRequest) {
  const token = request.headers.get('Authorization')?.replace('Bearer ', '');

  if (!token) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const site = await getSiteByToken(token);
  if (!site) {
    return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
  }

  const chatbots = await getChatbotsForUser(site.userId);

  // Consistent response format
  return NextResponse.json({
    success: true,
    data: chatbots.map(c => ({
      id: c.id,
      name: c.name,
      active: c.active,
      style: c.style,
      messagesTemplate: c.messagesTemplate,
    })),
  });
}
```

```php
<?php
// WordPress: includes/integration/class-api-handler.php
class API_Handler {
    private $api_base = 'https://app.aibotkit.io/api/wordpress';

    public function get_chatbots() {
        $token = get_option( 'ai_botkit_site_token' );

        $response = wp_remote_get(
            $this->api_base . '/chatbots',
            array(
                'headers' => array(
                    'Authorization' => 'Bearer ' . $token,
                    'Content-Type'  => 'application/json',
                ),
                'timeout' => 30,
            )
        );

        if ( is_wp_error( $response ) ) {
            return array( 'success' => false, 'error' => $response->get_error_message() );
        }

        $body = json_decode( wp_remote_retrieve_body( $response ), true );

        if ( ! isset( $body['success'] ) || ! $body['success'] ) {
            return array( 'success' => false, 'error' => $body['error'] ?? 'Unknown error' );
        }

        return $body;
    }
}
```

### 2. Authentication Flow

**Check for:**
- Token generation security
- Token storage (both sides)
- Token validation
- Token refresh/expiration

**SaaS Token Generation:**
```typescript
// src/app/api/wordpress/connect/route.ts
export async function POST(request: NextRequest) {
  const user = await getUser();
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { siteUrl } = await request.json();

  // Validate site URL
  const url = new URL(siteUrl);
  if (!['http:', 'https:'].includes(url.protocol)) {
    return NextResponse.json({ error: 'Invalid URL' }, { status: 400 });
  }

  // Generate secure token
  const token = crypto.randomUUID() + '-' + crypto.randomUUID();

  // Store site registration
  await db.insert(aibotkitSites).values({
    userId: user.id,
    siteUrl: url.origin,
    token: token,
  });

  return NextResponse.json({ token });
}
```

**WordPress Token Storage:**
```php
<?php
// Secure storage
update_option( 'ai_botkit_site_token', $token );

// Token retrieval with validation
function ai_botkit_get_token() {
    $token = get_option( 'ai_botkit_site_token', '' );

    if ( empty( $token ) ) {
        return new WP_Error( 'no_token', 'Site not connected' );
    }

    return $token;
}
```

### 3. Request/Response Format

**Check for:**
- Consistent JSON structure
- Error format standardization
- Status code usage
- Content-Type headers

**Standardized Response Format:**
```typescript
// Success response
{
  "success": true,
  "data": { /* response data */ }
}

// Error response
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE" // optional
}
```

**Status Code Usage:**
| Status | Meaning | When to Use |
|--------|---------|-------------|
| 200 | Success | Successful GET, PUT |
| 201 | Created | Successful POST creating resource |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid token |
| 403 | Forbidden | Valid token, no permission |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Unexpected error |

### 4. Error Handling

**Check for:**
- Graceful error handling on both sides
- User-friendly error messages
- Error logging
- Retry logic for transient failures

**WordPress Error Handling:**
```php
<?php
public function api_request( $endpoint, $method = 'GET', $body = null ) {
    $args = array(
        'method'  => $method,
        'headers' => array(
            'Authorization' => 'Bearer ' . $this->get_token(),
            'Content-Type'  => 'application/json',
        ),
        'timeout' => 30,
    );

    if ( $body ) {
        $args['body'] = wp_json_encode( $body );
    }

    $response = wp_remote_request( $this->api_base . $endpoint, $args );

    // Network error
    if ( is_wp_error( $response ) ) {
        $this->log_error( 'Network error: ' . $response->get_error_message() );
        return array(
            'success' => false,
            'error'   => __( 'Could not connect to AI BotKit. Please try again.', 'ai-botkit-for-lead-generation' ),
        );
    }

    $status_code = wp_remote_retrieve_response_code( $response );
    $body = json_decode( wp_remote_retrieve_body( $response ), true );

    // API error
    if ( $status_code >= 400 ) {
        $this->log_error( "API error {$status_code}: " . ( $body['error'] ?? 'Unknown' ) );

        if ( 401 === $status_code ) {
            return array(
                'success' => false,
                'error'   => __( 'Site connection expired. Please reconnect.', 'ai-botkit-for-lead-generation' ),
                'code'    => 'TOKEN_EXPIRED',
            );
        }

        return array(
            'success' => false,
            'error'   => $body['error'] ?? __( 'An error occurred.', 'ai-botkit-for-lead-generation' ),
        );
    }

    return $body;
}
```

### 5. Data Synchronization

**Check for:**
- Chatbot data consistency
- Cache invalidation
- Update propagation
- Conflict resolution

**Good Pattern:**
```php
<?php
// WordPress: Cache chatbot data with TTL
function ai_botkit_get_chatbot_cached( $chatbot_id ) {
    $cache_key = 'ai_botkit_chatbot_' . $chatbot_id;
    $cached = get_transient( $cache_key );

    if ( false !== $cached ) {
        return $cached;
    }

    $api = new API_Handler();
    $result = $api->get_chatbot( $chatbot_id );

    if ( $result['success'] ) {
        set_transient( $cache_key, $result['data'], HOUR_IN_SECONDS );
        return $result['data'];
    }

    return null;
}

// Invalidate on update webhook
function ai_botkit_handle_chatbot_updated( $chatbot_id ) {
    delete_transient( 'ai_botkit_chatbot_' . $chatbot_id );
}
```

### 6. Streaming Responses (Chat)

**Check for:**
- SSE format compliance
- Connection handling
- Timeout configuration
- Error recovery mid-stream

**SaaS Streaming:**
```typescript
// src/app/api/chat/route.ts
export async function POST(request: NextRequest) {
  // ... validation ...

  const stream = new ReadableStream({
    async start(controller) {
      try {
        for await (const chunk of ragEngine.streamResponse(messages)) {
          const data = `data: ${JSON.stringify({ content: chunk })}\n\n`;
          controller.enqueue(new TextEncoder().encode(data));
        }
        controller.enqueue(new TextEncoder().encode('data: [DONE]\n\n'));
      } catch (error) {
        const errorData = `data: ${JSON.stringify({ error: 'Stream error' })}\n\n`;
        controller.enqueue(new TextEncoder().encode(errorData));
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

## Output Format

```markdown
## API Integration Review

### Summary
| Area | Status | Issues |
|------|--------|--------|
| Endpoint Consistency | Status | X |
| Authentication | Status | X |
| Request/Response Format | Status | X |
| Error Handling | Status | X |
| Data Synchronization | Status | X |
| Streaming | Status | X |

### Issues Found

#### HIGH: Inconsistent Error Response Format
**SaaS File:** src/app/api/wordpress/chatbots/route.ts
**WordPress File:** includes/integration/class-api-handler.php

**Issue:** Error responses don't match expected format

**SaaS Returns:**
```json
{ "error": "Not found" }
```

**WordPress Expects:**
```json
{ "success": false, "error": "Not found" }
```

**Recommended Fix:** Standardize on success/error format
**Estimated Fix Time:** 30 minutes

---

### API Endpoint Matrix

| Endpoint | SaaS | WordPress | Status |
|----------|------|-----------|--------|
| GET /wordpress/chatbots | Yes | Yes | OK |
| POST /wordpress/connect | Yes | Yes | OK |
| POST /chat | Yes | Yes | OK |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (both components)
- Can be invoked directly for integration-focused review

## Related Agents

- `saas-security-auditor` - SaaS security
- `wordpress-security-auditor` - WordPress security
- `nextjs-standards-reviewer` - API route patterns
