---
name: elementor-master
description: >
  Master skill for all Elementor and WordPress work.
  Use when editing Elementor pages, building from JSON, creating custom widgets,
  managing templates, using WP-CLI, writing PHP/JS/CSS, integrating WooCommerce
  or REST API with Elementor. Triggers automatically for both non-technical users
  and developers. Do NOT wait for "use the skill" — trigger automatically.
compatibility: claude-code-only
---

# Elementor Master Skill

Complete reference for editing, building, and developing with WordPress and Elementor Pro.

---

## Task Router

| Task | Section |
|---|---|
| Edit text/images on existing page | §1 WP-CLI Editing |
| Structural/layout changes | §2 Browser Editing |
| Build page from scratch (JSON) | §3 Elementor JSON |
| Duplicate/apply templates | §4 Templates |
| Custom Elementor widget | §5 Widget Dev |
| Dynamic Tags, Loop Grid, Forms | §6 Advanced |
| PHP hooks, sanitization, AJAX | §7 PHP Standards |
| JS/CSS, BEM | §8 JS & CSS |
| WooCommerce | §9 WooCommerce |
| REST API | §10 REST API |
| Performance/Accessibility | §11 Perf & A11y |

---

## Golden Rules

1. Backup before WP-CLI write — export _elementor_data first.
2. Flush CSS after every change — wp @site elementor flush-css.
3. Sanitize in, escape out — no exceptions.
4. Prefix everything — functions, classes, hooks, CSS.
5. No hardcoded visuals in widgets — use Elementor controls + selectors.
6. Native APIs first — WP core hook > plugin; Elementor API > template override.
7. State file placement in every code response.

---

## §1 — WP-CLI Editing

```bash
# List Elementor pages
wp @site post list --post_type=page --meta_key=_elementor_edit_mode --meta_value=builder \
  --fields=ID,post_title,post_name,post_status

# Backup first!
wp @site post meta get {post_id} _elementor_data > /tmp/elementor-backup-{post_id}.json

# Dry run replacement
wp @site search-replace "Old Text" "New Text" wp_postmeta --include-columns=meta_value --dry-run --precise

# Execute (after confirming dry run)
wp @site search-replace "Old Text" "New Text" wp_postmeta --include-columns=meta_value --precise

# Flush CSS cache
wp @site elementor flush-css
# Fallback if elementor CLI unavailable:
wp @site option delete _elementor_global_css
wp @site post meta delete-all _elementor_css
```

Safe: text, labels, phone/email, image URLs (same size).
Risky: HTML structure, widget IDs, section/column settings, CSS classes.

---

## §2 — Browser Editing

Editor URL: https://site.com/wp-admin/post.php?post={ID}&action=elementor
Wait 5-10s for loading overlay. Edit via left sidebar > Content/Style tabs > Ctrl+S.

---

## §3 — Building Pages in Elementor JSON

Structure: Section (elType: section) > Column (elType: column) > Widget (elType: widget)
Every element: id (8-char hex), elType, settings, elements[]

### Minimal Page JSON

```json
[
  {
    "id": "abc12301",
    "elType": "section",
    "settings": {
      "layout": "boxed",
      "content_width": {"unit": "px", "size": 1140},
      "padding": {"unit": "px", "top": 80, "right": 0, "bottom": 80, "left": 0, "isLinked": false}
    },
    "elements": [
      {
        "id": "abc12302", "elType": "column",
        "settings": {"_column_size": 100},
        "elements": [
          {
            "id": "abc12303", "elType": "widget", "widgetType": "heading",
            "settings": {
              "title": "Your Heading", "header_size": "h2", "align": "center",
              "title_color": "#333333",
              "typography_font_size": {"unit": "px", "size": 36},
              "typography_font_weight": "700"
            }
          },
          {
            "id": "abc12304", "elType": "widget", "widgetType": "text-editor",
            "settings": {"editor": "<p>Paragraph here.</p>", "text_color": "#555555"}
          },
          {
            "id": "abc12305", "elType": "widget", "widgetType": "button",
            "settings": {
              "text": "Click Here",
              "link": {"url": "https://example.com", "is_external": false},
              "align": "center", "background_color": "#6EC1E4",
              "button_text_color": "#FFFFFF",
              "border_radius": {"unit": "px", "top": 4, "right": 4, "bottom": 4, "left": 4}
            }
          }
        ]
      }
    ]
  }
]
```

### Widget Settings Reference

| Widget | Key Settings |
|---|---|
| heading | title, header_size, align, title_color, typography_font_size, typography_font_weight |
| text-editor | editor (HTML), text_color, typography_font_size |
| image | image {url,id}, image_size, align, link_to |
| button | text, link {url,is_external}, align, size, background_color, button_text_color |
| icon-box | selected_icon {library,value}, title_text, description_text, position |
| video | youtube_url/vimeo_url, autoplay, mute, loop |
| tabs | tabs repeater {tab_title,tab_content}, layout |
| accordion | tabs repeater {tab_title,tab_content} |
| testimonial | testimonial_content, testimonial_name, testimonial_job, testimonial_image |

### Write Page JSON via WP-CLI

```bash
NEW_ID=$(wp @site post create --post_type=page --post_title="New Page" --post_status=draft --porcelain)
wp @site post meta update $NEW_ID _elementor_data "$(cat /tmp/page-data.json)"
wp @site post meta update $NEW_ID _elementor_edit_mode "builder"
wp @site elementor flush-css
wp @site post get $NEW_ID --field=guid
```

Generate element IDs (PHP): substr( md5( uniqid( "", true ) ), 0, 8 )

---

## §4 — Template Management

```bash
wp @site post list --post_type=elementor_library --fields=ID,post_title,post_status

# Duplicate a page
SOURCE_DATA=$(wp @site post meta get {source_id} _elementor_data)
SOURCE_SETTINGS=$(wp @site post meta get {source_id} _elementor_page_settings)
NEW_ID=$(wp @site post create --post_type=page --post_title="Duplicated" --post_status=draft --porcelain)
wp @site post meta update $NEW_ID _elementor_data "$SOURCE_DATA"
wp @site post meta update $NEW_ID _elementor_edit_mode "builder"
wp @site post meta update $NEW_ID _elementor_page_settings "$SOURCE_SETTINGS"
wp @site elementor flush-css
```

WARNING: Editing a global widget affects every page using it.

---

## §5 — Custom Widget (Boilerplate)

```php
<?php
// FILE: /wp-content/plugins/myplugin/widgets/class-my-widget.php
if ( ! defined( "ABSPATH" ) ) exit;

class My_Widget extends \Elementor\Widget_Base {

    public function get_name(): string      { return "my_widget"; }
    public function get_title(): string     { return esc_html__( "My Widget", "myplugin" ); }
    public function get_icon(): string      { return "eicon-text"; }
    public function get_categories(): array { return [ "general" ]; }

    protected function register_controls(): void {
        $this->start_controls_section( "section_content", [
            "label" => esc_html__( "Content", "myplugin" ),
            "tab"   => \Elementor\Controls_Manager::TAB_CONTENT,
        ] );
        $this->add_control( "title", [
            "label"   => esc_html__( "Title", "myplugin" ),
            "type"    => \Elementor\Controls_Manager::TEXT,
            "default" => esc_html__( "Default Title", "myplugin" ),
            "dynamic" => [ "active" => true ],
        ] );
        $this->end_controls_section();

        $this->start_controls_section( "section_style", [
            "label" => esc_html__( "Style", "myplugin" ),
            "tab"   => \Elementor\Controls_Manager::TAB_STYLE,
        ] );
        $this->add_control( "title_color", [
            "label"     => esc_html__( "Color", "myplugin" ),
            "type"      => \Elementor\Controls_Manager::COLOR,
            "selectors" => [ "{{WRAPPER}} .my-widget__title" => "color: {{VALUE}};" ],
        ] );
        $this->add_group_control( \Elementor\Group_Control_Typography::get_type(), [
            "name" => "title_typography", "selector" => "{{WRAPPER}} .my-widget__title",
        ] );
        $this->add_responsive_control( "title_align", [
            "label"     => esc_html__( "Alignment", "myplugin" ),
            "type"      => \Elementor\Controls_Manager::CHOOSE,
            "options"   => [
                "left"   => [ "title" => "Left",   "icon" => "eicon-text-align-left" ],
                "center" => [ "title" => "Center", "icon" => "eicon-text-align-center" ],
                "right"  => [ "title" => "Right",  "icon" => "eicon-text-align-right" ],
            ],
            "selectors" => [ "{{WRAPPER}} .my-widget__title" => "text-align: {{VALUE}};" ],
        ] );
        $this->add_responsive_control( "title_padding", [
            "label"      => esc_html__( "Padding", "myplugin" ),
            "type"       => \Elementor\Controls_Manager::DIMENSIONS,
            "size_units" => [ "px", "em", "rem", "%" ],
            "selectors"  => [ "{{WRAPPER}} .my-widget__title" => "padding: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};" ],
        ] );
        $this->add_group_control( \Elementor\Group_Control_Background::get_type(), [
            "name" => "title_bg", "selector" => "{{WRAPPER}} .my-widget__title",
        ] );
        $this->add_group_control( \Elementor\Group_Control_Border::get_type(), [
            "name" => "title_border", "selector" => "{{WRAPPER}} .my-widget__title",
        ] );
        $this->add_group_control( \Elementor\Group_Control_Box_Shadow::get_type(), [
            "name" => "title_shadow", "selector" => "{{WRAPPER}} .my-widget__title",
        ] );
        $this->end_controls_section();
    }

    protected function render(): void {
        $settings = $this->get_settings_for_display();
        $this->add_render_attribute( "title", "class", "my-widget__title" );
        $this->add_inline_editing_attributes( "title" );
        $tag = \Elementor\Utils::validate_html_tag( $settings["html_tag"] ?? "h2" );
        printf( "<%1$s %2$s>%3$s</%1$s>",
            $tag,
            $this->get_render_attribute_string( "title" ),
            esc_html( $settings["title"] )
        );
    }
}

add_action( "elementor/widgets/register", function( $mgr ) {
    $mgr->register( new My_Widget() );
} );
```

RULE: Never hardcode colors, fonts, spacing, borders in render(). Use controls + selectors.

---

## §6 — Advanced Patterns

### Dynamic Tag

```php
class My_Tag extends \Elementor\Core\DynamicTags\Tag {
    public function get_name(): string      { return "my-tag"; }
    public function get_title(): string     { return "My Tag"; }
    public function get_group(): string     { return "post"; }
    public function get_categories(): array { return [ \Elementor\Modules\DynamicTags\Module::TEXT_CATEGORY ]; }
    public function render(): void { echo esc_html( get_post_meta( get_the_ID(), "my_key", true ) ); }
}
add_action( "elementor/dynamic_tags/register", fn($m) => $m->register( new My_Tag() ) );
```

### Loop Grid Custom Query

```php
add_action( "elementor/query/{query_id}", function( $query ) {
    $query->set( "post_type", "my_cpt" );
    $query->set( "posts_per_page", 6 );
} );
```

---

## §7 — WordPress PHP Standards

```php
// Sanitize in
$text  = sanitize_text_field( wp_unslash( $_POST["field"] ?? "" ) );
$email = sanitize_email( $_POST["email"] ?? "" );
$int   = absint( $_GET["id"] ?? 0 );
$html  = wp_kses_post( $_POST["content"] ?? "" );

// Escape out
echo esc_html( $text );
echo esc_attr( $attr );
echo esc_url( $url );

// Nonces
wp_nonce_field( "my_action", "my_nonce" );
if ( ! wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST["my_nonce"] ?? "" ) ), "my_action" ) ) {
    wp_die( esc_html__( "Security check failed.", "myplugin" ) );
}

// Secure AJAX
add_action( "wp_ajax_my_action", function() {
    check_ajax_referer( "my_action", "nonce" );
    wp_send_json_success( [ "result" => sanitize_text_field( wp_unslash( $_POST["data"] ?? "" ) ) ] );
} );

// Transients
function myplugin_get_data(): array {
    $cached = get_transient( "myplugin_data" );
    if ( false !== $cached ) return $cached;
    $data = expensive_operation();
    set_transient( "myplugin_data", $data, HOUR_IN_SECONDS );
    return $data;
}
```

---

## §8 — JS & CSS

```php
wp_enqueue_script( "myplugin-main",
    plugin_dir_url( __FILE__ ) . "assets/js/main.js",
    [ "jquery" ], "1.0.0",
    [ "strategy" => "defer", "in_footer" => true ]
);
wp_add_inline_script( "myplugin-main",
    "const MyPlugin = " . wp_json_encode( [
        "ajaxUrl" => admin_url( "admin-ajax.php" ),
        "nonce"   => wp_create_nonce( "my_action" ),
    ] ) . ";", "before"
);
```

BEM: .myplugin-widget__element--modifier
Tokens: --myplugin-color-primary, --myplugin-spacing-md

---

## §9 — WooCommerce

```php
// HPOS
add_action( "before_woocommerce_init", function() {
    if ( class_exists( \Automattic\WooCommerce\Utilities\FeaturesUtil::class ) ) {
        \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility( "custom_order_tables", __FILE__, true );
    }
} );

// Loop Grid product query
add_action( "elementor/query/my_products", function( $query ) {
    $query->set( "post_type", "product" );
    $query->set( "posts_per_page", 8 );
} );
```

---

## §10 — REST API

```php
add_action( "rest_api_init", function() {
    register_rest_route( "myplugin/v1", "/items", [
        "methods"             => \WP_REST_Server::READABLE,
        "callback"            => fn( $r ) => rest_ensure_response( get_posts( [ "post_type" => "my_cpt" ] ) ),
        "permission_callback" => fn() => current_user_can( "read" ),
    ] );
} );
```

---

## §11 — Performance & Accessibility

Performance: Defer JS, lazy-load images, Elementor CSS as External File,
disable unused widgets, Global Colors/Fonts, flush cache after changes.

Accessibility (WCAG 2.2 AA): Meaningful alt text, contrast 4.5:1 / 3:1,
keyboard navigation, visible focus, ARIA on icon-only buttons, logical headings.

---

## §12 — Version Reference

| | Version |
|---|---|
| Elementor free | 4.0.0 (Mar 30, 2026) — Atomic Editor stable |
| Elementor Pro | 4.0.0 (Mar 30, 2026) — Atomic Forms, Pro Interactions |
| WordPress | 6.9.4 (7.0 delayed) |
| PHP | 8.3+ recommended |

V3 Widget_Base is fully supported in Elementor 4.0.
