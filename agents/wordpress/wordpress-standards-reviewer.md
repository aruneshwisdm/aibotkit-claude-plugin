# WordPress Standards Reviewer

Reviews WordPress plugin code for coding standards compliance, best practices, and common issues.

## Purpose

Ensure the AI BotKit WordPress plugin follows:
- WordPress Coding Standards (WPCS)
- WordPress security best practices
- Plugin architecture patterns
- Internationalization (i18n) standards
- Hook usage conventions

## When to Use

- During `/full-review` for WordPress plugin component
- When adding new PHP code
- Before submitting to WordPress.org
- After major refactoring

## Architecture Context

AI BotKit WordPress Plugin Structure:
```
wordpress-plugin/
├── ai-botkit-for-lead-generation.php  # Main plugin file
├── admin/
│   └── views/                          # Admin page templates
├── includes/
│   ├── class-ai-botkit.php            # Main plugin class
│   ├── admin/                          # Admin functionality
│   ├── integration/                    # REST API integration
│   └── public/                         # Frontend functionality
├── public/                             # Public assets
└── languages/                          # Translation files
```

## What Gets Analyzed

### 1. Coding Standards (WPCS)

**Check for:**
- Proper indentation (tabs, not spaces)
- Brace style (K&R)
- Naming conventions
- Yoda conditions
- Proper spacing

**Good Pattern:**
```php
<?php
/**
 * Get chatbot by ID.
 *
 * @param int $chatbot_id The chatbot ID.
 * @return array|null The chatbot data or null.
 */
function ai_botkit_get_chatbot( $chatbot_id ) {
    if ( empty( $chatbot_id ) ) {
        return null;
    }

    $chatbot = get_option( 'ai_botkit_chatbot_' . absint( $chatbot_id ) );

    if ( false === $chatbot ) {
        return null;
    }

    return $chatbot;
}
```

**Anti-Pattern:**
```php
<?php
// Missing docblock
// Wrong spacing
// Wrong naming convention
function getAiBotKitChatbot($chatbotId) {
    if($chatbotId == null){ // Wrong style, not Yoda
        return null;
    }
    return get_option('ai_botkit_chatbot_'.$chatbotId); // Missing sanitization
}
```

### 2. Security Practices

**Check for:**
- Nonce verification for forms
- Capability checks
- Data sanitization/escaping
- Direct file access prevention

**Good Pattern:**
```php
<?php
// Prevent direct access
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

/**
 * Handle settings save.
 */
function ai_botkit_save_settings() {
    // Verify nonce
    if ( ! isset( $_POST['ai_botkit_nonce'] ) ||
         ! wp_verify_nonce( $_POST['ai_botkit_nonce'], 'ai_botkit_settings' ) ) {
        wp_die( esc_html__( 'Security check failed', 'ai-botkit-for-lead-generation' ) );
    }

    // Check capability
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( esc_html__( 'Unauthorized', 'ai-botkit-for-lead-generation' ) );
    }

    // Sanitize input
    $api_key = sanitize_text_field( wp_unslash( $_POST['api_key'] ?? '' ) );

    // Save
    update_option( 'ai_botkit_api_key', $api_key );

    wp_redirect( add_query_arg( 'updated', 'true', wp_get_referer() ) );
    exit;
}
add_action( 'admin_post_ai_botkit_save_settings', 'ai_botkit_save_settings' );
```

**Anti-Pattern:**
```php
<?php
function ai_botkit_save_settings() {
    // No nonce check
    // No capability check
    $api_key = $_POST['api_key']; // No sanitization
    update_option( 'ai_botkit_api_key', $api_key );
}
```

### 3. Output Escaping

**Check for:**
- esc_html() for HTML content
- esc_attr() for attributes
- esc_url() for URLs
- wp_kses() for allowed HTML

**Good Pattern:**
```php
<div class="chatbot-wrapper" data-id="<?php echo esc_attr( $chatbot_id ); ?>">
    <h2><?php echo esc_html( $chatbot_name ); ?></h2>
    <a href="<?php echo esc_url( $chatbot_url ); ?>">
        <?php esc_html_e( 'View Chatbot', 'ai-botkit-for-lead-generation' ); ?>
    </a>
    <div class="description">
        <?php echo wp_kses_post( $description ); ?>
    </div>
</div>
```

**Anti-Pattern:**
```php
<div class="chatbot-wrapper" data-id="<?php echo $chatbot_id; ?>">
    <h2><?php echo $chatbot_name; ?></h2>
    <a href="<?php echo $chatbot_url; ?>">View Chatbot</a>
</div>
```

### 4. Internationalization (i18n)

**Check for:**
- All user-facing strings translatable
- Proper use of text domain
- Correct function usage (__(), _e(), esc_html__(), etc.)
- Placeholder handling

**Good Pattern:**
```php
<?php
// Simple string
$title = __( 'Chatbot Settings', 'ai-botkit-for-lead-generation' );

// Echo directly
_e( 'Save Changes', 'ai-botkit-for-lead-generation' );

// With escaping
echo esc_html__( 'Configure your chatbot', 'ai-botkit-for-lead-generation' );

// With placeholders
printf(
    /* translators: %s: chatbot name */
    esc_html__( 'Chatbot "%s" saved successfully', 'ai-botkit-for-lead-generation' ),
    esc_html( $chatbot_name )
);
```

**Anti-Pattern:**
```php
<?php
echo 'Chatbot Settings'; // Not translatable
echo __( 'Save', 'wrong-text-domain' ); // Wrong text domain
echo "Chatbot $name saved"; // Not translatable, variable in string
```

### 5. Hook Usage

**Check for:**
- Proper hook prefixing
- Correct priority/argument count
- Use of apply_filters for extensibility
- Proper action hooks for events

**Good Pattern:**
```php
<?php
// Prefixed hooks
add_action( 'init', 'ai_botkit_init' );
add_filter( 'ai_botkit_chatbot_data', 'ai_botkit_filter_chatbot_data', 10, 2 );

// Allow customization
$chatbot_data = apply_filters( 'ai_botkit_chatbot_data', $chatbot_data, $chatbot_id );

// Fire events
do_action( 'ai_botkit_chatbot_created', $chatbot_id, $chatbot_data );
```

### 6. REST API Integration

**Check for:**
- Proper endpoint registration
- Permission callbacks
- Input validation
- Response format

**Good Pattern:**
```php
<?php
/**
 * Register REST routes.
 */
function ai_botkit_register_rest_routes() {
    register_rest_route(
        'ai-botkit/v1',
        '/chatbots',
        array(
            'methods'             => WP_REST_Server::READABLE,
            'callback'            => 'ai_botkit_get_chatbots',
            'permission_callback' => function() {
                return current_user_can( 'edit_posts' );
            },
        )
    );
}
add_action( 'rest_api_init', 'ai_botkit_register_rest_routes' );
```

### 7. AJAX Handling

**Check for:**
- Nonce verification
- Capability checks
- Proper response format
- Error handling

**Good Pattern:**
```php
<?php
/**
 * Handle AJAX request.
 */
function ai_botkit_ajax_save_chatbot() {
    check_ajax_referer( 'ai_botkit_nonce', 'nonce' );

    if ( ! current_user_can( 'manage_options' ) ) {
        wp_send_json_error( array( 'message' => 'Unauthorized' ), 403 );
    }

    $chatbot_id = isset( $_POST['chatbot_id'] ) ? absint( $_POST['chatbot_id'] ) : 0;

    if ( empty( $chatbot_id ) ) {
        wp_send_json_error( array( 'message' => 'Invalid chatbot ID' ), 400 );
    }

    // Process...

    wp_send_json_success( array( 'chatbot_id' => $chatbot_id ) );
}
add_action( 'wp_ajax_ai_botkit_save_chatbot', 'ai_botkit_ajax_save_chatbot' );
```

## Output Format

```markdown
## WordPress Standards Review

### Summary
| Category | Issues | Severity |
|----------|--------|----------|
| Coding Standards | X | High/Medium/Low |
| Security | X | High/Medium/Low |
| Escaping | X | High/Medium/Low |
| Internationalization | X | High/Medium/Low |
| Hooks | X | High/Medium/Low |

### Issues Found

#### HIGH: Missing Nonce Verification
**File:** includes/admin/class-ajax-handler.php:45
**Issue:** AJAX handler missing nonce check

**Current:**
```php
function handle_save() {
    $data = $_POST['data'];
    // Process...
}
```

**Recommended:**
```php
function handle_save() {
    check_ajax_referer( 'ai_botkit_nonce', 'nonce' );
    $data = sanitize_text_field( wp_unslash( $_POST['data'] ?? '' ) );
    // Process...
}
```

**Impact:** CSRF vulnerability
**Estimated Fix Time:** 15 minutes

---

### WPCS Compliance

| Rule | Violations | Files |
|------|------------|-------|
| WordPress.Security.NonceVerification | X | X files |
| WordPress.Security.EscapeOutput | X | X files |
| WordPress.WP.I18n | X | X files |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (WordPress plugin component)
- Can be invoked directly for WordPress-focused review

## Related Agents

- `wordpress-security-auditor` - Deeper security analysis
- `api-integration-reviewer` - SaaS-Plugin integration
- `saas-security-auditor` - SaaS counterpart
