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

---

## §1 — Editing Existing Pages via WP-CLI

```bash
# List Elementor pages
wp @site post list --post_type=page \
  --meta_key=_elementor_edit_mode --meta_value=builder \
  --fields=ID,post_title,post_name,post_status

# 1. Backup
wp @site post meta get {post_id} _elementor_data > /tmp/elementor-backup-{post_id}.json

# 2. Dry run
wp @site search-replace "Old Text" "New Text" wp_postmeta \
  --include-columns=meta_value --dry-run --precise

# 3. Execute
wp @site search-replace "Old Text" "New Text" wp_postmeta \
  --include-columns=meta_value --precise

# 4. Flush CSS
wp @site elementor flush-css
# If CLI not available:
wp @site option delete _elementor_global_css
wp @site post meta delete-all _elementor_css

# 5. Verify
wp @site post get {post_id} --fields=ID,post_title,post_status,guid
```

Safe to replace: text, button labels, phone/email, image URLs (same size).
Risky: HTML structure, widget IDs, section/column settings, CSS classes.

---

## §2 — Editing via Browser Automation

Editor URL: https://example.com/wp-admin/post.php?post={ID}&action=elementor

Wait 5-10 seconds for loading overlay to disappear.

- Edit text: click element > Content tab > edit > Ctrl+S
- Change image: click image > thumbnail > Media Library > Insert Media > Save
- Edit button: click button > Content/Style tabs > Save

```bash
playwright-cli -s=wp-editor navigate "https://example.com/wp-admin/post.php?post={ID}&action=elementor"
```

---

## §3 — Building Pages in Elementor JSON

Modern Elementor (3.6+) uses **Containers with Flexbox** exclusively.
Structure: Container (outer, boxed) > Container (inner, row/column) > Widget
Every element needs: `id` (7-char random string), `elType`, `settings`, `elements` (children).

---

## §3a — Container Architecture (Modern Elementor)

⛔ **FORBIDDEN** — Never use these in generated JSON:
- `"elType": "section"`
- `"elType": "column"`

✅ **REQUIRED** — Use only:
- `"elType": "container"` for ALL layout wrappers

### Outer Container (full-width section wrapper)

```json
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
  "elements": []
}
```

### Inner Container (row — side by side)

```json
{
  "id": "e4f5g6h",
  "elType": "container",
  "settings": {
    "content_width": "full",
    "flex_direction": "row",
    "justify_content": "space-between",
    "align_items": "center",
    "gap": {"unit": "px", "size": 40, "column": "40", "row": "40", "isLinked": true},
    "width": {"unit": "%", "size": 100}
  },
  "elements": []
}
```

### Flexbox Settings Reference

| Setting | Values |
|---|---|
| `flex_direction` | `"row"` / `"column"` / `"row-reverse"` / `"column-reverse"` |
| `justify_content` | `"flex-start"` / `"center"` / `"flex-end"` / `"space-between"` / `"space-evenly"` |
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

---

## §3b — Animation Doctrine (Critical — Live Site Visibility)

### The Core Rule

⛔ **NEVER** animate a Container (`elType: "container"`) — this locks it at `opacity: 0` on the live site. Appears fine in editor but INVISIBLE to visitors.

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

✅ **ALLOWED**:
- `fadeInUp` — recommended default
- `fadeInRight` — from right side
- `zoomIn` — zoom from center
- `fadeIn` — simple fade

⛔ **FORBIDDEN** (cause horizontal scroll / layout jumps on RTL):
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

- **Required**: Hero widgets, bottom CTA widgets
- **Optional/subtle**: Service cards, testimonials — only on inner widgets
- **Never**: Container wrappers, background elements

---

## §3c — Import Wrapper (Always Required)

⛔ **FORBIDDEN** — Never output raw array:
```json
[ { "id": "...", "elType": "container", ... } ]
```

✅ **REQUIRED** — Always wrap in official Elementor envelope:

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

Import: Elementor → Templates → Saved Templates → Import Templates → select .json → Insert.

---

## §3d — Widget Settings Anti-Bug Rules

| Widget | ✅ Correct key | ⛔ Wrong key |
|---|---|---|
| text-editor content | `editor` | `content` (causes Lorem Ipsum) |
| Carousel slides | `slides` | `items` |
| Icons | `selected_icon: {library, value}` | `icon` alone |
| External links | `"is_external": "on"` | `true` / `false` |
| JSON IDs | 7-char random string | sequential numbers |

### Widget Quick Reference

**heading:** `title`, `header_size` (h1-h6), `align`, `title_color`, `typography_font_family`, `typography_font_size`, `typography_font_weight`, `_animation`, `_animation_delay`

**text-editor:** `editor` (HTML — NOT `content`), `text_color`, `typography_font_family`, `typography_font_size`

**button:** `text`, `link {url, is_external: "on"}`, `align`, `size`, `background_color`, `button_text_color`, `border_radius`, `_animation`

**icon-box:** `selected_icon {library: "fa-solid", value: "fas fa-star"}`, `title_text`, `description_text`, `icon_color`, `icon_size`

**testimonial-carousel (Pro):** `slides` array (NOT `items`), each: `{content, image {url}, name, title}`

**accordion:** `tabs` repeater: `tab_title`, `tab_content`

### Default Font Rule
Always use `"typography_font_family": "Heebo"` unless client specifies otherwise.

### Write JSON to Page via WP-CLI

```bash
NEW_ID=$(wp @site post create --post_type=page --post_title="New Page" --post_status=draft --porcelain)
wp @site post meta update $NEW_ID _elementor_data "$(cat /tmp/page-data.json)"
wp @site post meta update $NEW_ID _elementor_edit_mode "builder"
wp @site elementor flush-css
wp @site post get $NEW_ID --field=guid
```

Generate unique IDs: `substr( md5( uniqid( '', true ) ), 0, 7 )`

---

## §4 — Template Management

```bash
# List templates
wp @site post list --post_type=elementor_library --fields=ID,post_title,post_status

# Duplicate a page
SOURCE_DATA=$(wp @site post meta get {source_id} _elementor_data)
NEW_ID=$(wp @site post create --post_type=page --post_title="Duplicated" --post_status=draft --porcelain)
wp @site post meta update $NEW_ID _elementor_data "$SOURCE_DATA"
wp @site post meta update $NEW_ID _elementor_edit_mode "builder"
wp @site elementor flush-css
```

WARNING: Editing a global widget affects every page that uses it.

---

## §5 — Custom Widget Development

### Mandatory Output Format

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
        $this->end_controls_section();

        $this->start_controls_section( 'section_style', [
            'label' => esc_html__( 'Style', 'myplugin' ),
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
        $this->end_controls_section();
    }

    protected function render(): void {
        $settings = $this->get_settings_for_display();
        $this->add_render_attribute( 'title', 'class', 'my-widget__title' );
        $tag = \Elementor\Utils::validate_html_tag( $settings['html_tag'] ?? 'h2' );
        printf( '<%1$s %2$s>%3$s</%1$s>',
            $tag,
            $this->get_render_attribute_string( 'title' ),
            esc_html( $settings['title'] )
        );
    }
}
add_action( 'elementor/widgets/register', function( $mgr ) {
    $mgr->register( new My_Widget() );
} );
```

### Controls Checklist — Never Hardcode Visuals
Typography, color, background, border, border-radius, box-shadow, padding, margin, alignment, hover — ALL via Elementor controls with selectors.

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
$text  = sanitize_text_field( wp_unslash( $_POST['field'] ?? '' ) );
$email = sanitize_email( $_POST['email'] ?? '' );
$int   = absint( $_GET['id'] ?? 0 );
$html  = wp_kses_post( $_POST['content'] ?? '' );
$url   = esc_url_raw( $_POST['url'] ?? '' );

echo esc_html( $text );
echo esc_attr( $attr );
echo esc_url( $url );
echo wp_kses_post( $html );
```

---

## §8 — JS & CSS Standards

```php
wp_enqueue_script( 'myplugin-main',
    plugin_dir_url( __FILE__ ) . 'assets/js/main.js',
    [ 'jquery' ], '1.0.0',
    [ 'strategy' => 'defer', 'in_footer' => true ]
);
```

BEM: .myplugin-widget__element--modifier
Design tokens: --myplugin-color-primary, --myplugin-spacing-md

---

## §9 — WooCommerce

```php
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

Elementor 4.0.0 + Pro 4.0.0 (March 2026) — Atomic Editor stable, V3 Widget_Base fully supported.
WordPress 6.9.4 current stable. PHP 8.3+ recommended.
Container support since Elementor 3.6 (2022). Sections/columns deprecated in Atomic Editor.

---

## §13 — Homepage Architecture (Standard Layout)

Standard section order (do not change unless briefed):

1. **Hero** — H1, subtitle, CTA. Min-height 80vh. Animations on all widgets.
2. **About (short)** — Split: text + image in row container.
3. **Services** — Rich cards: icon, title, description, box-shadow, border-radius.
4. **Why Us** — Split layout with icons/values.
5. **Gallery** — Image grid.
6. **Testimonials** — Carousel, min 3 shown.
7. **FAQ** — Accordion, min 5 Q&As.
8. **Bottom CTA** — Closing section. Animations required. WhatsApp CTA.

### CTA Convention (WhatsApp First)
All CTA buttons: `https://wa.me/972XXXXXXXXX`
`"link": {"url": "https://wa.me/972500000000", "is_external": "on"}`

Contact details appear ONLY in Footer and Contact page — never in content sections.

---

## §3e — Grid Layout Rules (Items per Row)

Apply these ALWAYS for card grids, services, stats, or any repeating element.

### The Rules

| Item count | Layout | Width each |
|---|---|---|
| 1 | Full width | 100% |
| 2 | 1 row of 2 | 48% |
| 3 | 1 row of 3 | 31% |
| 4 | **1 row of 4 — NEVER 3+1** | 23% |
| 5 | 3 top + 2 bottom centered | 31% / 31% |
| 6 | 2 rows of 3 | 31% |
| 8 | 2 rows of 4 | 23% |
| 9 | 3 rows of 3 | 31% |

### Key Rules (User Preference)
- 4 items = always 1 row of 4. NEVER 3+1 — looks broken.
- 5 items = 3+2, NEVER 4+1 — symmetry always wins.
- 6 items = 2 rows of 3.

### 4-Card Row

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
  "elements": ["card1(23%)", "card2(23%)", "card3(23%)", "card4(23%)"]
}
```

### 5-Card Layout (3+2) — Two Separate Row Containers

Row 1: 3 cards at 31% each, justify_content: center
Row 2: 2 cards at 31% each, justify_content: center

---

## §14 — UI/UX Design Principles

Integrated from the frontend-design skill. Apply to every Elementor page.

### Pre-Build Direction Check

1. Purpose: what problem does this page solve? who is the visitor?
2. Tone: luxury/refined · bold/energetic · minimal/clean · warm · tech/futuristic
3. One Memorable Element: the single visual detail the visitor will remember

### Typography Scale

| Element | Size | Weight | Line-height |
|---|---|---|---|
| H1 Hero | 52-70px | 900 | 1.1em |
| H2 Section | 36-48px | 700-900 | 1.15em |
| H3 Card title | 20-24px | 700 | 1.3em |
| Body text | 16-18px | 400 | 1.7em |
| Labels/tags | 12-14px | 600 | — |

- Default font: Heebo for Hebrew. Never use Arial, Inter, Roboto.
- Weight contrast: 900 hero titles + 400 body = strong visual rhythm.

### Color Rules

- Max 3 main colors: 1 primary + 1 accent + neutrals.
- Body text contrast: minimum 4.5:1 (WCAG AA).
- Dark bg values: #07080f / #0f1120 / #1a1a2e — not pure #000.
- Light text on dark: #ffffff headings · #8891b4 subtext.
- Accent: CTA buttons + highlights only — not everything.

### Spacing

- Section padding: 80-100px top/bottom minimum.
- Card internal padding: 28-36px.
- Gap between cards: 20-32px.
- Hero min-height: 80vh.
- Generous whitespace = trust signal.

### Visual Depth for Dark Cards

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

### Section Background Alternation

- Hero: #07080f
- Stats bar: #07080f + border #1e2240
- Services: #07080f
- Process: #0f1120
- Testimonials: #07080f
- CTA: gradient #0f0820 to #07080f

### Final Quality Checklist

- [ ] Every section: 80px+ padding top/bottom
- [ ] Card counts obey §3e — no orphan cards (no 3+1, no 4+1)
- [ ] Animations only on widgets — NEVER on containers
- [ ] All CTAs: WhatsApp + is_external: "on"
- [ ] Font: Heebo + correct size hierarchy
- [ ] Dark cards: border + box-shadow
- [ ] No text-only sections — always visual structure
- [ ] Animation delays staggered 100-200ms apart
