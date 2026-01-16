# WordPress Security Auditor

Performs comprehensive security audit of the WordPress plugin focusing on common vulnerabilities and WordPress-specific security practices.

## Purpose

Identify security vulnerabilities in the AI BotKit WordPress plugin including:
- SQL injection
- Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)
- Insecure Direct Object References (IDOR)
- Authentication/Authorization bypasses
- File inclusion vulnerabilities
- Insecure data storage

## When to Use

- During `/full-review` for WordPress plugin component
- Before releasing plugin updates
- After adding user input handling
- When handling sensitive data

## What Gets Analyzed

### 1. SQL Injection

**Check for:**
- Use of $wpdb->prepare() for all queries
- No direct variable interpolation in SQL
- Proper escaping of LIKE clauses

**Good Pattern:**
```php
<?php
global $wpdb;

// Using prepare with placeholders
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}ai_botkit_sessions WHERE user_id = %d AND status = %s",
        $user_id,
        $status
    )
);

// LIKE clause with proper escaping
$search = $wpdb->esc_like( $search_term ) . '%';
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}ai_botkit_chatbots WHERE name LIKE %s",
        $search
    )
);
```

**Anti-Pattern:**
```php
<?php
// Direct variable interpolation - SQL INJECTION!
$results = $wpdb->get_results(
    "SELECT * FROM {$wpdb->prefix}ai_botkit_sessions WHERE user_id = {$user_id}"
);

// String concatenation - SQL INJECTION!
$results = $wpdb->query(
    "DELETE FROM sessions WHERE id = " . $_GET['id']
);
```

### 2. Cross-Site Scripting (XSS)

**Check for:**
- All output escaped with appropriate function
- User input never directly echoed
- wp_kses() for allowed HTML

**Good Pattern:**
```php
<?php
// Plain text
echo esc_html( $user_input );

// Attribute
echo '<input value="' . esc_attr( $value ) . '">';

// URL
echo '<a href="' . esc_url( $url ) . '">Link</a>';

// JavaScript
echo '<script>var data = ' . wp_json_encode( $data ) . ';</script>';

// Allowed HTML
echo wp_kses_post( $content );

// Custom allowed HTML
$allowed = array(
    'a' => array( 'href' => array(), 'title' => array() ),
    'br' => array(),
    'strong' => array(),
);
echo wp_kses( $content, $allowed );
```

**Anti-Pattern:**
```php
<?php
// Direct output - XSS!
echo $_GET['message'];
echo $user_name;
echo "<div>{$content}</div>";
```

### 3. Cross-Site Request Forgery (CSRF)

**Check for:**
- Nonce fields in all forms
- Nonce verification before processing
- Proper nonce action naming

**Good Pattern:**
```php
<?php
// Form with nonce
?>
<form method="post" action="">
    <?php wp_nonce_field( 'ai_botkit_save_settings', 'ai_botkit_nonce' ); ?>
    <input type="text" name="api_key" value="<?php echo esc_attr( $api_key ); ?>">
    <button type="submit">Save</button>
</form>

<?php
// Processing with verification
if ( isset( $_POST['ai_botkit_nonce'] ) ) {
    if ( ! wp_verify_nonce( $_POST['ai_botkit_nonce'], 'ai_botkit_save_settings' ) ) {
        wp_die( 'Security check failed' );
    }
    // Process form...
}

// AJAX nonce verification
function ai_botkit_ajax_handler() {
    check_ajax_referer( 'ai_botkit_ajax_nonce', 'nonce' );
    // Process...
}
```

**Anti-Pattern:**
```php
<?php
// Form without nonce - CSRF!
?>
<form method="post">
    <input type="text" name="api_key">
    <button type="submit">Save</button>
</form>

<?php
// Processing without verification
if ( isset( $_POST['api_key'] ) ) {
    update_option( 'api_key', $_POST['api_key'] ); // CSRF vulnerable!
}
```

### 4. Capability Checks

**Check for:**
- current_user_can() before sensitive operations
- Appropriate capability for operation
- Checks on both display and processing

**Good Pattern:**
```php
<?php
// Admin page access
function ai_botkit_admin_page() {
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( __( 'Unauthorized access', 'ai-botkit-for-lead-generation' ) );
    }
    // Render page...
}

// AJAX with capability check
function ai_botkit_delete_chatbot() {
    check_ajax_referer( 'ai_botkit_nonce', 'nonce' );

    if ( ! current_user_can( 'manage_options' ) ) {
        wp_send_json_error( 'Unauthorized', 403 );
    }

    // Process delete...
}

// Menu registration with capability
add_menu_page(
    'AI BotKit',
    'AI BotKit',
    'manage_options', // Capability required
    'ai-botkit',
    'ai_botkit_admin_page'
);
```

**Anti-Pattern:**
```php
<?php
// No capability check - unauthorized access!
function ai_botkit_admin_page() {
    // Anyone can access if they know the URL
    // Render sensitive admin page...
}
```

### 5. Data Sanitization

**Check for:**
- sanitize_text_field() for text
- absint() for integers
- sanitize_email() for emails
- wp_unslash() before sanitization

**Good Pattern:**
```php
<?php
// Text input
$name = sanitize_text_field( wp_unslash( $_POST['name'] ?? '' ) );

// Integer
$id = absint( $_POST['id'] ?? 0 );

// Email
$email = sanitize_email( wp_unslash( $_POST['email'] ?? '' ) );

// URL
$url = esc_url_raw( wp_unslash( $_POST['url'] ?? '' ) );

// Textarea (preserves newlines)
$content = sanitize_textarea_field( wp_unslash( $_POST['content'] ?? '' ) );

// Array of integers
$ids = array_map( 'absint', (array) ( $_POST['ids'] ?? array() ) );
```

**Anti-Pattern:**
```php
<?php
// No sanitization - security risk!
$name = $_POST['name'];
$id = $_POST['id'];
```

### 6. File Operations

**Check for:**
- File type validation
- Use of wp_upload_dir()
- Proper permissions
- Path traversal prevention

**Good Pattern:**
```php
<?php
// Validate file type
$allowed_types = array( 'image/jpeg', 'image/png', 'image/gif' );
$file_type = wp_check_filetype( $_FILES['file']['name'] );

if ( ! in_array( $file_type['type'], $allowed_types, true ) ) {
    wp_die( 'Invalid file type' );
}

// Use WordPress upload directory
$upload_dir = wp_upload_dir();
$target_dir = $upload_dir['basedir'] . '/ai-botkit/';

// Prevent directory traversal
$filename = sanitize_file_name( $_FILES['file']['name'] );
$target_path = $target_dir . $filename;

// Verify path is within allowed directory
if ( strpos( realpath( dirname( $target_path ) ), realpath( $target_dir ) ) !== 0 ) {
    wp_die( 'Invalid path' );
}
```

### 7. REST API Security

**Check for:**
- Permission callbacks on all routes
- Input validation
- Output sanitization

**Good Pattern:**
```php
<?php
register_rest_route( 'ai-botkit/v1', '/chatbots/(?P<id>\d+)', array(
    'methods'             => 'GET',
    'callback'            => 'ai_botkit_get_chatbot',
    'permission_callback' => function( $request ) {
        return current_user_can( 'edit_posts' );
    },
    'args'                => array(
        'id' => array(
            'validate_callback' => function( $param ) {
                return is_numeric( $param );
            },
            'sanitize_callback' => 'absint',
        ),
    ),
) );
```

**Anti-Pattern:**
```php
<?php
register_rest_route( 'ai-botkit/v1', '/chatbots/(?P<id>\d+)', array(
    'methods'             => 'GET',
    'callback'            => 'ai_botkit_get_chatbot',
    'permission_callback' => '__return_true', // Public access to sensitive data!
) );
```

## Output Format

```markdown
## WordPress Security Audit

### Risk Summary

| Risk Level | Count | Categories |
|------------|-------|------------|
| CRITICAL | X | SQL Injection, Auth Bypass |
| HIGH | X | XSS, CSRF |
| MEDIUM | X | Missing Sanitization |
| LOW | X | Informational |

### Vulnerabilities Found

#### CRITICAL: SQL Injection in Chatbot Query
**File:** includes/class-ai-botkit.php:156
**Type:** SQL Injection
**OWASP:** A03:2021 - Injection

**Vulnerable Code:**
```php
$wpdb->get_results( "SELECT * FROM chatbots WHERE id = {$_GET['id']}" );
```

**Fixed Code:**
```php
$wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}ai_botkit_chatbots WHERE id = %d",
        absint( $_GET['id'] )
    )
);
```

**Impact:** Database compromise, data theft
**Estimated Fix Time:** 15 minutes

---

### Security Checklist

| Check | Status | Files Affected |
|-------|--------|----------------|
| SQL Injection Protected | Pass/Fail | X files |
| XSS Prevented | Pass/Fail | X files |
| CSRF Protected | Pass/Fail | X files |
| Capability Checks | Pass/Fail | X files |
| Data Sanitized | Pass/Fail | X files |
| Nonces Verified | Pass/Fail | X files |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (WordPress plugin component)
- Can be invoked directly for security-focused review

## Related Agents

- `wordpress-standards-reviewer` - Coding standards
- `saas-security-auditor` - SaaS security
- `api-integration-reviewer` - API security
