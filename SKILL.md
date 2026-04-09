---
name: elementor-master
description: >
  Master skill for all Elementor and WordPress work — editing, building, and developing.
  Use this whenever the user asks to: edit an Elementor page, update text or images,
  build a new page from scratch, write Elementor JSON data, create custom widgets,
  use Dynamic Tags, work with Theme Builder, write PHP/JS/CSS for Elementor, manage
  templates, work with WP-CLI on Elementor data, use Loop Grids, integrate ACF,
  WooCommerce, or REST API with Elementor, or debug any WordPress/Elementor issue.
  Trigger for both non-technical users and developers. Do NOT wait for the user
  to say "use the skill" — trigger automatically.
compatibility: claude-code-only
---

# Elementor Master Skill

Complete reference for editing, building, and developing with WordPress and Elementor Pro.
Covers everything from a simple text update via WP-CLI to building custom widgets,
writing raw Elementor JSON, and integrating with ACF, WooCommerce, and REST API.

---

## Task Router

| What the user wants | Go to section |
|---|---|
| Edit text, images, buttons on existing page | §1 Editing via WP-CLI |
| Structural changes, layout, sections | §2 Editing via Browser |
| Build page from scratch programmatically | §3 Building Pages in JSON |
| Container structure & Flexbox rules | §3a Container Architecture |
| Animation rules (live site visibility) | §3b Animation Doctrine |
| Import wrapper format | §3c Import Wrapper |
| Duplicate or apply templates | §4 Template Management |
| Create custom Elementor widget | §5 Custom Widget Development |
| Dynamic Tags, Loop Grid, Form actions | §6 Advanced Patterns |
| PHP hooks, sanitization, AJAX | §7 WordPress PHP Standards |
| JS/CSS enqueue, BEM | §8 JS & CSS Standards |
| WooCommerce integration | §9 WooCommerce |
| REST API endpoint | §10 REST API |
| Performance / Accessibility | §11 Performance & Accessibility |

---

## Golden Rules

1. Backup before any WP-CLI write — export _elementor_data before touching it.
2. Flush CSS cache after every change — wp @site elementor flush-css.
3. Sanitize in, escape out — every input sanitized, every output escaped.
4. Prefix everything — functions, classes, constants, hooks, CSS classes.
5. No hardcoded visuals in widgets — every color, font, spacing MUST be an Elementor control with selectors. Never inline styles in render().
6. Native APIs first — WordPress core hook before plugin; Elementor API before template override.
7. State placement in every code response — declare exactly which file.
8. No over-clarifying — state assumption and proceed unless it would materially change the code.
9. **Containers Only (JSON)** — NEVER use `"elType": "section"` or `"elType": "column"` in generated JSON. Modern Elementor uses `"elType": "container"` exclusively.
10. **Never animate containers** — Animating a container causes `opacity:0` on the live site. Animations go ONLY on widgets inside containers.
11. **Always wrap JSON for import** — Raw arrays cannot be imported. Always use the version/title/type/content envelope.
12. **Never use `flex_wrap: "wrap"`** — It is NOT respected on import. Always split multi-row grids into separate explicit `row_nowrap` containers (see §3e).
13. **Accordion/Toggle widget — safe settings only** — Only these keys are safe: `tabs`, `active_item`, `title_html_tag`, `title_color`, `active_color`, `content_color`. Any other style keys (background_color, icon{}, icon_active{}, border_color, typography_*, icon_color) break the widget editor panel on import. Note: active title color key is `active_color`, NOT `title_active_color`.

---

## §1 — Editing Existing Pages via WP-CLI/ `"flex-end"` / `"space-between"` / `"space-around"` / `"space-evenly"` |
| `align_items` | `"flex-start"` / `"center"` / `"flex-end"` / `"stretch"` |
| `flex_wrap` | `"wrap"` / `"nowrap"` |

### Hero Container (min-height 80vh)

```json
{
  "id": "m1n2o3p",
  "elType": "container",
  "settings": {
    "content_width": "boxed",
    "boxed_width": {"unit": "px", "size": 1200},
    "flex_direction": "column",
    "justify_content": "center",
    "align_items": "center",
    "min_height": {"unit": "vh", "size": 80},
    "padding": {"unit": "px", "top": "80", "bottom": "80", "left": "0", "right": "0", "isLinked": false},
    "background_background": "classic",
    "background_color": "#07080f"
  },
  "elements": []
}
```

### Complete Minimal Page Example (Container-based)

```json
{
  "version": "0.4",
  "title": "My Page",
  "type": "page",
  "content": [
    {
      "id": "a1b2c3d",
      "elType": "container",
      "settings": {
        "content_width": "boxed",
        "boxed_width": {"unit": "px", "size": 1200},
        "flex_direction": "column",
        "justify_content": "center",
        "align_items": "center",
        "padding": {"unit": "px", "top": "80", "bottom": "80", "left": "0", "right": "0", "isLinked": false},
        "background_background": "classic",
        "background_color": "#ffffff"
      },
      "elements": [
        {
          "id": "e4f5g6h",
          "elType": "widget",
          "widgetType": "heading",
          "settings": {
            "title": "כותרת ראשית",
            "header_size": "h1",
            "align": "center",
            "title_color": "#1a1a2e",
            "typography_typography": "custom",
            "typography_font_family": "Heebo",
            "typography_font_size": {"unit": "px", "size": 52},
            "typography_font_weight": "700",
            "_animation": "fadeInUp",
            "animation_duration": "slow",
            "_animation_delay": 200
          }
        },
        {
          "id": "i7j8k9l",
          "elType": "widget",
          "widgetType": "text-editor",
          "settings": {
            "editor": "<p>תיאור קצר ומשכנע של הפרויקט</p>",
            "text_color": "#555555",
            "typography_typography": "custom",
            "typography_font_family": "Heebo",
            "typography_font_size": {"unit": "px", "size": 18},
            "_animation": "fadeInUp",
            "animation_duration": "slow",
            "_animation_delay": 400
          }
        },
        {
          "id": "m1n2o3p",
          "elType": "widget",
          "widgetType": "button",
          "settings": {
            "text": "צור קשר בוואטסאפ",
            "link": {"url": "https://wa.me/972500000000", "is_external": "on"},
            "align": "center",
            "background_color": "#25D366",
            "button_text_color": "#FFFFFF",
            "border_radius": {"unit": "px", "top": 50, "right": 50, "bottom": 50, "left": 50, "isLinked": true},
            "typography_font_family": "Heebo",
            "typography_font_weight": "700",
            "_animation": "zoomIn",
            "animation_duration": "slow",
            "_animation_delay": 600
          }
        }
      ]
    }
  ]
}
```

---

## §3b — Animation Doctrine (Critical — Live Site Visibility)

### The Core Rule

⛔ **NEVER** animate a Container (`elType: "container"`) — this locks the container at `opacity: 0` on the live site. It appears fine in the editor but is invisible to visitors.

✅ **ONLY** animate Widgets (headings, text-editor, button, icon-box, image) inside containers.

### Animation Syntax (Correct)

```json
{
  "elType": "widget",
  "widgetType": "heading",
  "settings": {
    "title": "כותרת",
    "_animation": "fadeInUp",
    "animation_duration": "slow",
    "_animation_delay": 200
  }
}
```

Note: `_animation` uses a leading underscore. `animation_duration` does NOT.

### RTL-Safe Animations (Hebrew/Arabic sites)

✅ **ALLOWED** — these do not cause horizontal scroll glitches on RTL:
- `fadeInUp` — enters from below (recommended default)
- `fadeInRight` — enters from right
- `zoomIn` — zoom from center
- `fadeIn` — simple fade

⛔ **FORBIDDEN** — these cause layout jumps on RTL sites:
- `fadeInLeft`
- `slideInLeft`
- `bounceInLeft`

### Animation Delay Strategy

```
Hero H1:          _animation_delay: 0
Hero subtitle:    _animation_delay: 200
Hero CTA button:  _animation_delay: 400
Section cards:    _animation_delay: 100 per card (100, 200, 300...)
Bottom CTA:       _animation_delay: 0
```

### When to Use Animation

- **Required**: Hero section widgets, bottom CTA section widgets
- **Optional/subtle**: Service cards, testimonials, stats — only on inner widgets
- **Never**: Section/container wrappers, background elements

---

## §3c — Import Wrapper (Always Required)

⛔ **FORBIDDEN** — Never output a raw array as the top level:
```json
[ { "id": "...", "elType": "container", ... } ]
```

✅ **REQUIRED** — Always wrap in the official Elementor import envelope:

```json
{
  "version": "0.4",
  "title": "Page Title — Project Name",
  "type": "page",
  "content": [
    // All containers go here
  ]
}
```

Import path in Elementor: Templates → Saved Templates → Import Templates → select .json file → Insert.

---

## §3d — Widget Settings Anti-Bug Rules

| Widget | ✅ Correct key | ⛔ Wrong key |
|---|---|---|
| text-editor content | `editor` | `content` (causes Lorem Ipsum) |
| Carousel/testimonials slides | `slides` | `items` |
| Icons | `selected_icon: {library, value}` | `icon` alone |
| External links | `"is_external": "on"` | `true` / `false` |
| JSON IDs | 7-char random string | sequential numbers |

### Widget Settings Quick Reference

**heading:** `title`, `header_size` (h1-h6), `align`, `title_color`, `typography_font_family`, `typography_font_size {unit,size}`, `typography_font_weight`, `_animation`, `_animation_delay`

**text-editor:** `editor` (HTML string — NOT `content`), `text_color`, `typography_font_family`, `typography_font_size`

**button:** `text`, `link {url, is_external: "on"}`, `align`, `size` (xs/sm/md/lg/xl), `background_color`, `button_text_color`, `border_radius`, `typography_font_family`, `_animation`

**image:** `image {url, id}`, `image_size`, `align`, `link_to` (none/file/custom)

**icon-box:** `selected_icon {library: "fa-solid", value: "fas fa-star"}`, `title_text`, `description_text`, `position` (top/left/right), `icon_color`, `icon_size`

**testimonial-carousel (Pro):** `slides` (array — NOT `items`), each slide: `{content, image {url}, name, title}`

**accordion:** `tabs` (repeater: `tab_title`, `tab_content`)
⛔ **CRITICAL — Accordion import rule**: Only the following keys are safe in settings:

| Key | Example | Notes |
|-----|---------|-------|
| `tabs` | `[{_id, tab_title, tab_content}]` | Required |
| `active_item` | `1` | Required |
| `title_html_tag` | `"h3"` | Required |
| `title_color` | `"#ffffff"` | Safe — title text color |
| `active_color` | `"#00e5ff"` | Safe — open-item title color (⚠️ NOT `title_active_color`) |
| `content_color` | `"#94a3b8"` | Safe — answer text color |

**The following keys BREAK the widget editor panel on import** (accordion becomes non-editable):  
`background_color`, `icon{}`, `icon_active{}`, `border_color`, `typography_*`, `content_typography_*`, `icon_color`, `icon_active_color`, `title_active_color`.

```json
{
  "widgetType": "accordion",
  "settings": {
    "tabs": [
      { "_id": "a1b2c3d", "tab_title": "Question?", "tab_content": "<p>Answer text.</p>" }
    ],
    "active_item": 1,
    "title_html_tag": "h3",
    "title_color": "#ffffff",
    "active_color": "#00e5ff",
    "content_color": "#94a3b8"
  },
  "elements": []
}
```
For any other styling (border, background, typography, icons) — style inside the Elementor editor after import.

**video:** `youtube_url` / `vimeo_url`, `autoplay`, `mute`, `loop`

### Default Font Rule

Always use `"typography_font_family": "Heebo"` as the default font for all widgets unless the client specifies otherwise.

### Write JSON to Page via WP-CLI

```bash
NEW_ID=$(wp @site post create --post_type=page --post_title="New Page" --post_status=draft --porcelain)
wp @site post meta update $NEW_ID _elementor_data "$(cat /tmp/page-data.json)"
wp @site post meta update $NEW_ID _elementor_edit_mode "builder"
wp @site elementor flush-css
wp @site post get $NEW_ID --field=guid
```

Generate unique IDs in PHP: `substr( md5( uniqid( '', true ) ), 0, 7 )`

---

## §4 — Template Management

```bash
# List templates
wp @site post list --post_type=elementor_library --fields=ID,post_title,post_status

# Duplicate a page
SOURCE_DATA=$(wp @site post meta get {source_id} _elementor_data)
SOURCE_SETTINGS=$(wp @site post meta get {source_id} _elementor_page_settings)
NEW_ID=$(wp @site post create --post_type=page --post_title="Duplicated" --post_status=draft --porcelain)
wp @site post meta update $NEW_ID _elementor_data "$SOURCE_DATA"
wp @site post meta update $NEW_ID _elementor_edit_mode "builder"
wp @site post meta update $NEW_ID _elementor_page_settings "$SOURCE_SETTINGS"
wp @site elementor flush-css

# List global widgets
wp @site post list --post_type=elementor_library \
  --meta_key=_elementor_template_type --meta_value=widget --fields=ID,post_title
```

WARNING: Editing a global widget affects every page that uses it.

---

## §5 — Custom Widget Development

### Mandatory Output Format (every code response)

```
PLACEMENT — exact file path
REQUIRES — WordPress X.X+ | PHP X.X+ | Elementor X.X+
WHY — one paragraph on approach
CODE — complete, no truncation
INTEGRATION — manual steps if needed
```

### Widget Boilerplate

```php
<?php
// FILE: /wp-content/plugins/myplugin/widgets/class-my-widget.php
if ( ! defined( 'ABSPATH' ) ) exit;

class My_Widget extends \Elementor\Widget_Base {

    public function get_name(): string      { return 'my_widget'; }
    public function get_title(): string     { return esc_html__( 'My Widget', 'myplugin' ); }
    public function get_icon(): string      { return 'eicon-text'; }
    public function get_categories(): array { return [ 'general' ]; }

    protected function register_controls(): void {

        // ── Content ──
        $this->start_controls_section( 'section_content', [
            'label' => esc_html__( 'Content', 'myplugin' ),
            'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
        ] );
        $this->add_control( 'title', [
            'label'   => esc_html__( 'Title', 'myplugin' ),
            'type'    => \Elementor\Controls_Manager::TEXT,
            'default' => esc_html__( 'Default Title', 'myplugin' ),
            'dynamic' => [ 'active' => true ],
        ] );
        $this->add_control( 'html_tag', [
            'label'   => esc_html__( 'HTML Tag', 'myplugin' ),
            'type'    => \Elementor\Controls_Manager::SELECT,
            'options' => [ 'h1'=>'H1','h2'=>'H2','h3'=>'H3','h4'=>'H4','div'=>'div','p'=>'p' ],
            'default' => 'h2',
        ] );
        $this->end_controls_section();

        // ── Style ──
        $this->start_controls_section( 'section_style', [
            'label' => esc_html__( 'Title', 'myplugin' ),
            'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
        ] );
        $this->add_control( 'title_color', [
            'label'     => esc_html__( 'Color', 'myplugin' ),
            'type'      => \Elementor\Controls_Manager::COLOR,
            'selectors' => [ '{{WRAPPER}} .my-widget__title' => 'color: {{VALUE}};' ],
        ] );
        $this->add_group_control( \Elementor\Group_Control_Typography::get_type(), [
            'name' => 'title_typography', 'selector' => '{{WRAPPER}} .my-widget__title',
        ] );
        $this->add_responsive_control( 'title_align', [
            'label'     => esc_html__( 'Alignment', 'myplugin' ),
            'type'      => \Elementor\Controls_Manager::CHOOSE,
            'options'   => [
                'left'   => [ 'title' => 'Left',   'icon' => 'eicon-text-align-left' ],
                'center' => [ 'title' => 'Center', 'icon' => 'eicon-text-align-center' ],
                'right'  => [ 'title' => 'Right',  'icon' => 'eicon-text-align-right' ],
            ],
            'selectors' => [ '{{WRAPPER}} .my-widget__title' => 'text-align: {{VALUE}};' ],
        ] );
        $this->add_responsive_control( 'title_padding', [
            'label'      => esc_html__( 'Padding', 'myplugin' ),
            'type'       => \Elementor\Controls_Manager::DIMENSIONS,
            'size_units' => [ 'px', 'em', 'rem', '%' ],
            'selectors'  => [
                '{{WRAPPER}} .my-widget__title' =>
                    'padding: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
            ],
        ] );
        $this->add_group_control( \Elementor\Group_Control_Background::get_type(), [
            'name' => 'title_background', 'selector' => '{{WRAPPER}} .my-widget__title',
        ] );
        $this->add_group_control( \Elementor\Group_Control_Border::get_type(), [
            'name' => 'title_border', 'selector' => '{{WRAPPER}} .my-widget__title',
        ] );
        $this->add_group_control( \Elementor\Group_Control_Box_Shadow::get_type(), [
            'name' => 'title_box_shadow', 'selector' => '{{WRAPPER}} .my-widget__title',
        ] );
        $this->end_controls_section();
    }

    protected function render(): void {
        $settings = $this->get_settings_for_display();
        $this->add_render_attribute( 'title', 'class', 'my-widget__title' );
        $this->add_inline_editing_attributes( 'title' );
        $tag = \Elementor\Utils::validate_html_tag( $settings['html_tag'] ?? 'h2' );
        printf( '<%1$s %2$s>%3$s</%1$s>',
            $tag,
            $this->get_render_attribute_string( 'title' ),
            esc_html( $settings['title'] )
        );
    }
}

// Register:
add_action( 'elementor/widgets/register', function( $mgr ) {
    $mgr->register( new My_Widget() );
} );
```

### Controls Checklist (MANDATORY — never hardcode visuals)

Typography, text color, background, border, border-radius, box-shadow, padding, margin, alignment, hover states — ALL must be Elementor controls with selectors.

---

## §6 — Advanced Patterns

### Dynamic Tag

```php
class My_Dynamic_Tag extends \Elementor\Core\DynamicTags\Tag {
    public function get_name(): string      { return 'my-tag'; }
    public function get_title(): string     { return esc_html__( 'My Tag', 'myplugin' ); }
    public function get_group(): string     { return 'post'; }
    public function get_categories(): array { return [ \Elementor\Modules\DynamicTags\Module::TEXT_CATEGORY ]; }
    public function render(): void {
        echo esc_html( get_post_meta( get_the_ID(), 'my_meta_key', true ) );
    }
}
add_action( 'elementor/dynamic_tags/register', fn($m) => $m->register( new My_Dynamic_Tag() ) );
```

### Loop Grid Query

```php
add_action( 'elementor/query/{query_id}', function( $query ) {
    $query->set( 'post_type', 'my_cpt' );
    $query->set( 'posts_per_page', 6 );
} );
```

### Elementor Pro vs Free

| Feature | Free | Pro |
|---|---|---|
| Basic widgets | YES | YES |
| Theme Builder | NO | YES |
| Form widget | NO | YES |
| Dynamic Tags | NO | YES |
| Loop Grid | NO | YES |

---

## §7 — WordPress PHP Standards

```php
// Sanitize in
$text  = sanitize_text_field( wp_unslash( $_POST['field'] ?? '' ) );
$email = sanitize_email( $_POST['email'] ?? '' );
$int   = absint( $_GET['id'] ?? 0 );
$html  = wp_kses_post( $_POST['content'] ?? '' );
$url   = esc_url_raw( $_POST['url'] ?? '' );

// Escape out
echo esc_html( $text );
echo esc_attr( $attr );
echo esc_url( $url );
echo wp_kses_post( $html );

// Nonce
wp_nonce_field( 'my_action', 'my_nonce' );
if ( ! wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['my_nonce'] ?? '' ) ), 'my_action' ) ) {
    wp_die( esc_html__( 'Security check failed.', 'myplugin' ) );
}

// AJAX
add_action( 'wp_ajax_my_action', function() {
    check_ajax_referer( 'my_action', 'nonce' );
    wp_send_json_success( [ 'result' => sanitize_text_field( wp_unslash( $_POST['data'] ?? '' ) ) ] );
} );

// Transients
function myplugin_get_data(): array {
    $cached = get_transient( 'myplugin_data' );
    if ( false !== $cached ) return $cached;
    $data = expensive_operation();
    set_transient( 'myplugin_data', $data, HOUR_IN_SECONDS );
    return $data;
}
```

---

## §8 — JS & CSS Standards

```php
// Enqueue with defer
wp_enqueue_script( 'myplugin-main',
    plugin_dir_url( __FILE__ ) . 'assets/js/main.js',
    [ 'jquery' ], '1.0.0',
    [ 'strategy' => 'defer', 'in_footer' => true ]
);
wp_add_inline_script( 'myplugin-main',
    'const MyPlugin = ' . wp_json_encode( [
        'ajaxUrl' => admin_url( 'admin-ajax.php' ),
        'nonce'   => wp_create_nonce( 'my_action' ),
    ] ) . ';', 'before'
);
```

BEM naming: .myplugin-widget__element--modifier
Design tokens: --myplugin-color-primary, --myplugin-spacing-md

---

## §9 — WooCommerce

```php
// HPOS declaration
add_action( 'before_woocommerce_init', function() {
    if ( class_exists( \Automattic\WooCommerce\Utilities\FeaturesUtil::class ) ) {
        \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility( 'custom_order_tables', __FILE__, true );
    }
} );

// Loop Grid product query
add_action( 'elementor/query/my_products', function( $query ) {
    $query->set( 'post_type', 'product' );
    $query->set( 'posts_per_page', 8 );
} );
```

---

## §10 — REST API

```php
add_action( 'rest_api_init', function() {
    register_rest_route( 'myplugin/v1', '/items', [
        'methods'             => \WP_REST_Server::READABLE,
        'callback'            => fn( $r ) => rest_ensure_response( get_posts( [ 'post_type' => 'my_cpt' ] ) ),
        'permission_callback' => fn() => current_user_can( 'read' ),
    ] );
} );
```

---

## §11 — Performance & Accessibility

Performance: Defer JS, lazy-load images, Elementor CSS as External File, disable unused widgets, Global Colors/Fonts, flush cache after changes.

Accessibility (WCAG 2.2 AA): Meaningful alt text, contrast 4.5:1 (normal) / 3:1 (large), keyboard navigation, visible focus, ARIA on icon-only buttons, logical heading hierarchy.

---

## §12 — Version Reference

Elementor free 4.0.0 + Elementor Pro 4.0.0 (March 30, 2026) — Atomic Editor stable, V3 Widget_Base fully supported.
WordPress 6.9.4 current stable. PHP 8.3+ recommended.

Container support introduced in Elementor 3.6 (2022). All JSON generation must use containers — sections/columns are deprecated and removed in the Atomic Editor.

---

## §13 — Homepage Architecture (Standard Layout)

Standard section order for a homepage (do not change unless briefed otherwise):

1. **Hero** — H1, subtitle, CTA button. Min-height 80vh. Animations required on all widgets.
2. **About (short)** — Split layout: text left + image right (or vice versa). Inner row container.
3. **Services** — Rich cards with icon, title, description, box-shadow, border-radius.
4. **Why Us** — Split layout with icons/values.
5. **Gallery** — Image grid.
6. **Testimonials** — Carousel, minimum 3 testimonials shown.
7. **FAQ** — Accordion widget, minimum 5 Q&As.
8. **Bottom CTA** — Closing section. Animations required. WhatsApp CTA button.

### CTA Convention (WhatsApp First)
All CTA buttons link to WhatsApp: `https://wa.me/972XXXXXXXXX`
`"link": {"url": "https://wa.me/972500000000", "is_external": "on"}`

Contact details (phone, email, social) appear ONLY in Footer and Contact page — never scattered through content sections.

---

## §3e — Grid Layout Rules (Items per Row)

These rules determine how many inner containers (cards) to place per row. Apply these ALWAYS when building card grids, service sections, stats rows, or any repeating element layout.

### ⛔ CRITICAL: `flex_wrap: "wrap"` is BROKEN in Elementor JSON import

**Never use `flex_wrap: "wrap"` to build multi-row grids.** On import, Elementor ignores this setting and reverts to a single column. **Confirmed from live testing.**

✅ **The only correct approach**: create **separate explicit `row_nowrap` containers**, one per row.

```
// WRONG — will render as single column on import:
row_container(flex_wrap: "wrap") → [card, card, card, card, card, card]

// CORRECT — 2 rows of 3, proven to work:
row_container(flex_wrap: "nowrap") → [card1, card2, card3]
row_container(flex_wrap: "nowrap") → [card4, card5, card6]
```

### The Rules

| Item count | Layout | Inner container widths | Row structure |
|---|---|---|---|
| 1 | Full width | `100%` | 1 row |
| 2 | 2 columns | `48%` each | 1 `row_nowrap` |
| 3 | 1 row of 3 | `30%` each | 1 `row_nowrap` |
| **4** | **1 row of 4** — NEVER 3+1 | **`22%` each** | 1 `row_nowrap` |
| 5 | 3+2 (centered) | `30%` / `46%` | 2 `row_nowrap` |
| 6 | 2 rows of 3 | `30%` each | 2 `row_nowrap` |
| 8 | 2 rows of 4 | `22%` each | 2 `row_nowrap` |
| 9 | 3 rows of 3 | `30%` each | 3 `row_nowrap` |

### The Key Rule (User Preference)
> **4 items = always 1 row of 4.** Never 3+1 — that looks unbalanced.  
> **5 items = 3+2, never 4+1** — always prefer symmetry.  
> **6 items = 3+3, never 6 in one row.**

### Implementation — 4 cards in 1 row

```json
{
  "id": "row4abc",
  "elType": "container",
  "settings": {
    "content_width": "full",
    "flex_direction": "row",
    "justify_content": "space-between",
    "align_items": "stretch",
    "flex_wrap": "nowrap",
    "gap": {"unit": "px", "size": 20, "isLinked": true},
    "width": {"unit": "%", "size": 100}
  },
  "elements": [
    { "card 1 — width: 23%" },
    { "card 2 — width: 23%" },
    { "card 3 — width: 23%" },
    { "card 4 — width: 23%" }
  ]
}
```

Each card inner container:
```json
{
  "id": "card1xyz",
  "elType": "container",
  "settings": {
    "content_width": "full",
    "flex_direction": "column",
    "align_items": "flex-start",
    "width": {"unit": "%", "size": 23},
    "padding": {"unit": "px", "top": "32", "bottom": "32", "left": "28", "right": "28", "isLinked": false}
  }
}
```

### Implementation — 5 cards in 3+2 layout

Use two separate row containers — one with 3 cards, one with 2 cards (centered):

```json
[
  {
    "id": "row3abc",
    "elType": "container",
    "settings": {
      "content_width": "full",
      "flex_direction": "row",
      "justify_content": "center",
      "gap": {"unit": "px", "size": 24, "isLinked": true}
    },
    "elements": ["card1 (31%)", "card2 (31%)", "card3 (31%)"]
  },
  {
    "id": "row2def",
    "elType": "container",
    "settings": {
      "content_width": "full",
      "flex_direction": "row",
      "justify_content": "center",
      "gap": {"unit": "px", "size": 24, "isLinked": true}
    },
    "elements": ["card4 (31%)", "card5 (31%)"]
  }
]
```

---

## §14 — UI/UX Design Principles for Elementor Pages

Extracted and adapted from the `frontend-design` skill. Apply these when building any page in JSON or via the editor.

### Design Direction — Choose Before You Build

Before writing any JSON, define:
- **Purpose**: What problem does this page solve? Who is the visitor?
- **Tone**: Pick one: luxury/refined, bold/energetic, minimal/clean, warm/approachable, tech/futuristic
- **One Memorable Thing**: What's the single visual element the visitor will remember?

### Typography Rules

- **Default font**: `Heebo` for all Hebrew content
- **Heading sizes**: H1 = 52-70px · H2 = 36-48px · H3 = 22-28px · Body = 16-18px
- **Line height**: Headings 1.1-1.2em · Body text 1.65-1.75em
- **Weight contrast**: Mix 900 (hero titles) with 400 (body) — avoid all-700 pages
- **Never use**: Arial, Inter, Roboto as primary fonts

### Color System

- **Commit to a palette**: 1 primary + 1 accent + neutrals. No more than 3 main colors.
- **Contrast for text**: minimum 4.5:1 on all body text
- **Dark backgrounds**: use `#07080f` / `#0f1120` / `#1a1a2e` — avoid pure black `#000`
- **Light text on dark**: `#ffffff` for headings, `#8891b4` / `#94a3b8` for subtext
- **Accent usage**: CTA buttons, highlights, gradient text — not for every element

### Spacing & Rhythm

- **Section padding**: 80-100px top/bottom minimum
- **Card padding**: 28-36px internal
- **Gap between cards**: 20-32px
- **Hero min-height**: 80vh
- **Never cram**: generous whitespace signals quality and trust

### Visual Depth Techniques (for dark-theme pages)

Use these in container/card settings to create depth without images:

```json
{
  "border_border": "solid",
  "border_width": {"unit": "px", "top": "1", "right": "1", "bottom": "1", "left": "1", "isLinked": true},
  "border_color": "#1e2240",
  "border_radius": {"unit": "px", "top": 16, "right": 16, "bottom": 16, "left": 16, "isLinked": true},
  "box_shadow_box_shadow_type": "yes",
  "box_shadow_box_shadow": {"horizontal": 0, "vertical": 8, "blur": 30, "spread": 0, "color": "rgba(0,0,0,0.3)"}
}
```

For gradient top-border accent on cards:
```json
"background_background": "classic",
"background_color": "#0f1120",
"custom_css": ".elementor-element-WRAPPER { position: relative; } .elementor-element-WRAPPER::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 2px; background: linear-gradient(90deg, #7c3aed, #00e5ff); border-radius: 16px 16px 0 0; }"
```

### Section Visual Alternation

Alternate background colors between sections for visual rhythm:
- Hero: `#07080f` (darkest)
- Stats: `#07080f` + top/bottom border `#1e2240`
- Services: `#07080f`
- Process: `#0f1120` (slightly lighter)
- Testimonials: `#07080f`
- CTA: gradient `#0f0820` → `#07080f`

### Quality Checklist Before Finalizing JSON

- [ ] Every section has 80px+ top/bottom padding
- [ ] Card counts follow grid rules (§3e) — no orphan cards
- [ ] Animations only on widgets (never containers)
- [ ] All CTA buttons use WhatsApp link with `is_external: "on"`
- [ ] Typography uses Heebo + correct size hierarchy
- [ ] Dark sections have border + box-shadow on cards
- [ ] No section has only text — every section has visual structure (cards/icons/split)
- [ ] Staggered animation delays (100ms apart per card)
