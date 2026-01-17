# WordPress Hooks Analyzer Agent

Specialized agent for discovering and documenting all WordPress hooks (actions and filters) registered by the AI BotKit WordPress plugin.

## Purpose

Analyze the WordPress plugin to:
- Index all registered actions and filters
- Document hook priorities and callbacks
- Identify extension points for customization
- Detect potential hook conflicts
- Map plugin lifecycle through hooks

## When to Use

This agent is invoked by `/next-phase` during Phase 0.1 (Codebase Discovery):
- Understanding plugin architecture
- Planning new features that need hooks
- Debugging hook-related issues
- Documenting extension points

## What Gets Analyzed

### 1. Action Hooks

**Pattern Detection:**
```php
// Look for these patterns:
add_action('hook_name', 'callback', priority, accepted_args);
add_action('hook_name', [$this, 'method'], priority);
add_action('hook_name', [__CLASS__, 'static_method']);
do_action('custom_hook', $args);
```

**Categories:**
| Category | Hooks to Find |
|----------|---------------|
| Initialization | `init`, `plugins_loaded`, `wp_loaded` |
| Admin | `admin_menu`, `admin_init`, `admin_enqueue_scripts` |
| Frontend | `wp_enqueue_scripts`, `wp_footer`, `wp_head` |
| AJAX | `wp_ajax_*`, `wp_ajax_nopriv_*` |
| REST API | `rest_api_init` |
| Custom | `aibotkit_*` |

### 2. Filter Hooks

**Pattern Detection:**
```php
// Look for these patterns:
add_filter('hook_name', 'callback', priority, accepted_args);
apply_filters('custom_filter', $value, $args);
```

**Categories:**
| Category | Hooks to Find |
|----------|---------------|
| Content | `the_content`, `the_title` |
| Query | `pre_get_posts`, `posts_where` |
| Options | `option_*`, `pre_option_*` |
| Custom | `aibotkit_*` |

### 3. Custom Hooks (Extension Points)

**Plugin's Own Hooks:**
```php
// Actions the plugin provides for extension
do_action('aibotkit_before_chat_init', $chatbot_id);
do_action('aibotkit_after_message_sent', $message, $response);

// Filters the plugin provides
apply_filters('aibotkit_chat_response', $response, $chatbot_id);
apply_filters('aibotkit_widget_styles', $styles);
```

## Output Format

```markdown
# WordPress Plugin Hooks Analysis

Plugin: AI BotKit for Lead Generation
Version: X.X.X
Generated: [Date]

## Summary

| Type | Count | Custom Hooks |
|------|-------|--------------|
| Actions Registered | XX | XX |
| Filters Registered | XX | XX |
| Custom Actions Provided | XX | - |
| Custom Filters Provided | XX | - |

---

## Actions Registered

### Initialization Phase

| Hook | Callback | Priority | File:Line |
|------|----------|----------|-----------|
| `plugins_loaded` | `AI_BotKit::init` | 10 | `ai-botkit.php:45` |
| `init` | `AI_BotKit::register_shortcodes` | 10 | `includes/class-public.php:23` |

### Admin Phase

| Hook | Callback | Priority | File:Line |
|------|----------|----------|-----------|
| `admin_menu` | `AI_BotKit_Admin::add_menu` | 10 | `admin/class-admin.php:34` |
| `admin_init` | `AI_BotKit_Admin::register_settings` | 10 | `admin/class-admin.php:56` |
| `admin_enqueue_scripts` | `AI_BotKit_Admin::enqueue_assets` | 10 | `admin/class-admin.php:78` |

### Frontend Phase

| Hook | Callback | Priority | File:Line |
|------|----------|----------|-----------|
| `wp_enqueue_scripts` | `AI_BotKit_Public::enqueue_scripts` | 10 | `includes/class-public.php:45` |
| `wp_footer` | `AI_BotKit_Public::render_widget` | 99 | `includes/class-public.php:67` |

### AJAX Handlers

| Hook | Callback | Auth Required | File:Line |
|------|----------|---------------|-----------|
| `wp_ajax_aibotkit_send_message` | `handle_message` | No | `includes/class-ajax.php:23` |
| `wp_ajax_nopriv_aibotkit_send_message` | `handle_message` | No | `includes/class-ajax.php:23` |

### REST API

| Hook | Callback | File:Line |
|------|----------|-----------|
| `rest_api_init` | `AI_BotKit_REST::register_routes` | `includes/class-rest.php:12` |

---

## Filters Registered

| Hook | Callback | Priority | File:Line |
|------|----------|----------|-----------|
| `the_content` | `maybe_append_chatbot` | 99 | `includes/class-public.php:89` |
| `script_loader_tag` | `add_async_attribute` | 10 | `includes/class-public.php:101` |

---

## Custom Hooks Provided (Extension Points)

### Actions

| Hook | Parameters | Purpose | File:Line |
|------|------------|---------|-----------|
| `aibotkit_before_chat_init` | `$chatbot_id` | Before widget initializes | `includes/class-widget.php:34` |
| `aibotkit_after_message_sent` | `$message, $response` | After API response | `includes/class-ajax.php:67` |
| `aibotkit_lead_captured` | `$lead_data` | When lead is captured | `includes/class-leads.php:45` |

### Filters

| Hook | Parameters | Return | Purpose | File:Line |
|------|------------|--------|---------|-----------|
| `aibotkit_chat_response` | `$response, $chatbot_id` | `string` | Modify chat response | `includes/class-ajax.php:78` |
| `aibotkit_widget_styles` | `$styles` | `array` | Custom widget CSS | `includes/class-widget.php:56` |
| `aibotkit_allowed_pages` | `$pages` | `array` | Restrict display pages | `includes/class-public.php:23` |

---

## Hook Execution Order

```
plugins_loaded
    └── AI_BotKit::init()
            │
init        │
    └── register_shortcodes()
    └── register_post_types()
            │
admin_menu (if admin)
    └── add_menu_pages()
            │
wp_enqueue_scripts (if frontend)
    └── enqueue_frontend_assets()
            │
wp_footer   │
    └── render_chatbot_widget()
            └── do_action('aibotkit_before_chat_init')
```

---

## Potential Issues

### Priority Conflicts
- `the_content` filter at priority 99 may conflict with other plugins

### Missing Unhooks
- No `remove_action` calls found for cleanup

### Recommendations
1. Add `aibotkit_` prefix to all custom hooks for namespace safety
2. Document all custom hooks in readme.txt
3. Consider lower priority for `the_content` filter
```

## Integration

### Invoked By

- `/next-phase` command (Phase 0.1)
- `/full-review --component wordpress`

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:wordpress:wordpress-hooks-analyzer"
```

## Related Agents

- `code-capability-indexer` - Overall codebase indexing
- `wordpress-standards-reviewer` - WPCS compliance
- `wordpress-security-auditor` - Security review
