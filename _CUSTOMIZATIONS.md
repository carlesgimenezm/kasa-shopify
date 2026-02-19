# Theme Customizations

This file documents every custom change made to the Horizon theme.
Use this as a checklist when applying a new Horizon update.

Base theme: **Shopify Horizon** (https://github.com/Shopify/horizon)
Last merged: February 2026

---

## How to apply a new Horizon update

1. Download the new theme zip from Shopify admin
2. Unzip it alongside this repo
3. For each item below, re-apply the change to the new version
4. Or use git:
   ```bash
   git checkout -b horizon-update-vX.X
   # copy new theme files over, then:
   git diff main  # see what changed vs your customized version
   ```

---

## 1. Custom Font — ITC Garamond Std Light Narrow (h1 & h2)

**Font files (licensed):**
- `assets/ITCGaramondStd-LtNarrow.woff2`
- `assets/ITCGaramondStd-LtNarrow.woff`

**`snippets/fonts.liquid`** — Add at the very top, before the `{%- unless settings.type_body_font.system? -%}` line:

```liquid
{%- comment -%} Preload ITC Garamond Std Light Narrow for h1/h2 {%- endcomment -%}
<link
  rel="preload"
  as="font"
  href="{{ 'ITCGaramondStd-LtNarrow.woff2' | asset_url }}"
  type="font/woff2"
  crossorigin
  fetchpriority="high"
>
<style>
  @font-face {
    font-family: 'ITCGaramondStd-LtNarrow';
    src: url('{{ 'ITCGaramondStd-LtNarrow.woff2' | asset_url }}') format('woff2'),
         url('{{ 'ITCGaramondStd-LtNarrow.woff' | asset_url }}') format('woff');
    font-weight: normal;
    font-style: normal;
    font-display: swap;
  }
</style>
```

**`snippets/theme-styles-variables.liquid`** — After the `--font-heading--weight` line (around line 166), add:

```css
/* Custom h1/h2 font: ITC Garamond Std Light Narrow */
--font-h1--family: 'ITCGaramondStd-LtNarrow', Georgia, serif;
--font-h2--family: 'ITCGaramondStd-LtNarrow', Georgia, serif;
```

> **Why this works:** Horizon's `base.css` already uses `var(--font-h1--family)` and
> `var(--font-h2--family)` for all h1/h2 elements. These vars are never set by the
> theme itself, so defining them here is clean and won't conflict with theme updates.

---

## 2. Grind Size Selector — buy-buttons block

**File:** `blocks/buy-buttons.liquid`

This is a **fully custom block** — the entire file is replaced with our version.
The stock Horizon `buy-buttons.liquid` has no grind-related code whatsoever.

**What it adds:**
- A `<select>` dropdown for grind size, injected as a cart line item property
- A compact mobile variant that sits inline with the Add to Cart button
- Configurable via Theme Editor settings (no code changes needed per product)

**Schema settings added (at end of schema):**
```json
{ "type": "checkbox", "id": "enable_grind_dropdown", "label": "Show grind selector (main)", "default": true },
{ "type": "text", "id": "grind_label", "label": "Label", "default": "Grind size" },
{ "type": "text", "id": "grind_property_label", "label": "Property name (shown on order)", "default": "Grind Size" },
{ "type": "textarea", "id": "grind_options", "label": "Options (comma-separated)", "default": "Whole Bean, Espresso, Filter, French Press" },
{ "type": "checkbox", "id": "grind_required", "label": "Required", "default": true },
{ "type": "checkbox", "id": "enable_grind_compact_mobile", "label": "Compact dropdown on mobile (inside buttons row)", "default": true }
```

**How to re-apply after an update:**
Horizon rarely changes `buy-buttons.liquid` structurally. Do a diff between the
new stock version and ours to see if anything needs merging. The grind code is
self-contained — search for `{% if block_settings.enable_grind_dropdown %}`.

---

## 3. Product Section Custom Background — product-information section

**File:** `sections/product-information.liquid`

**What it adds:** Lets you set a custom background color or image per product page
directly in the Theme Editor, instead of being locked to the color scheme.

**Rendering code** — Insert after the `<script type="application/ld+json">` block (around line 3–5):

```liquid
{%- comment -%}
  Background priority:
  1) Background image (if enabled and set)
  2) Custom solid color
  3) Color scheme background
{%- endcomment -%}
{% if section.settings.use_bg_image and section.settings.bg_image != blank %}
  {% assign bg_size = section.settings.bg_image_fit %}
  {% case bg_size %}
    {% when 'cover' %}
      {% assign bg_size_value = 'cover' %}
    {% when 'contain' %}
      {% assign bg_size_value = 'contain' %}
    {% else %}
      {% assign bg_size_value = 'auto' %}
  {% endcase %}
  <div
    class="section-background section-background--image"
    style="
      background-image: url({{ section.settings.bg_image | image_url: width: 2000 }});
      background-size: {{ bg_size_value }};
      background-position: {{ section.settings.bg_image_position }};
      background-repeat: no-repeat;
    "
  ></div>
{% elsif section.settings.use_custom_bg and section.settings.custom_bg_solid != blank %}
  <div class="section-background" style="background-color: {{ section.settings.custom_bg_solid }};"></div>
{% endif %}
```

**Schema settings** — Add before the `color_scheme` setting in the schema:

```json
{ "type": "checkbox", "id": "use_custom_bg", "label": "Enable custom background color", "default": false },
{ "type": "color", "id": "custom_bg_solid", "label": "Custom background color", "default": "#ffffff" },
{ "type": "checkbox", "id": "use_bg_image", "label": "Enable background image", "default": false },
{ "type": "image_picker", "id": "bg_image", "label": "Background image" },
{ "type": "select", "id": "bg_image_fit", "label": "Image fit", "options": [{"value":"cover","label":"Cover"},{"value":"contain","label":"Contain"},{"value":"auto","label":"Original size"}], "default": "cover" },
{ "type": "select", "id": "bg_image_position", "label": "Image position", "default": "center center" }
```

> **Note:** `product-information.liquid` is the most-changed file in Horizon updates.
> Always diff carefully. The custom_bg code sits at the top of the file, clearly marked.

---

## 4. Global Settings — config/settings_data.json

This file is your live store configuration. **Never overwrite this with a new theme's
default** — always merge. Preserve your version and only add new keys from updates.

### Keys that differ from Horizon defaults:

| Key | Our value | Horizon default | Notes |
|-----|-----------|-----------------|-------|
| `cart_price_font` | `"secondary"` | `"subheading"` | Intentional brand choice |
| `logo_height` | `28` | *(added in newer Horizon)* | Keep as-is |
| `logo_height_mobile` | `24` | *(added in newer Horizon)* | Keep as-is |

### Color schemes (9 custom):
All 9 use brand colors (`#41261c`, `#752e09`) and Geist fonts.
Scheme IDs: `scheme-1` through `scheme-6`, plus three UUID-based schemes.

---

## 5. Global Custom CSS — platform_customizations

Stored in `settings_data.json` → `platform_customizations.custom_css[]`.
Survives theme code updates automatically (lives in settings, not code).

```css
.shopify-policy__title { text-align: left; margin-top: 100px; margin-bottom: 40px; }
.quick-add__button { border-radius: 8px; }
.unit-price { display: none; }
.button { font-family: Geist Mono; }
.product-card__badge--coming-soon, .product-badges__badge {
  background-color: #3f2c23; font-family: "Geist Mono"; font-size: 10px;
  border-radius: 4px; padding: 4px; line-height: 12px; color: white; text-transform: uppercase;
}
:root { --sidebar-width: 30rem; }
:root { --brand: #752e09; }
input[type="radio"] { accent-color: var(--brand); }
.shopify_subscriptions_app_policy { font-size: 14px; }
```

---

## 6. Page Layouts — templates/*.json

All template JSONs contain configured page layouts. **Always keep your versions.**
Never use the new theme's blank defaults.

Key templates:
- `templates/product.json` — Main product layout with grind settings
- `templates/product.accesorios-con-cafe.json`
- `templates/product.accesorios-sin-cafe.json`
- `templates/index.json` — Homepage
- `templates/page.shop.json`

---

## 7. Header & Footer Configuration

- `sections/header-group.json` — Logo heights, header layout
- `sections/footer-group.json` — FAQ accordion, email signup, contact info

---

## Files safe to replace wholesale on update

No customizations — take directly from new Horizon release:

- `assets/*.js` and `assets/*.css` (keep `ITCGaramondStd-LtNarrow.*`)
- `snippets/*.liquid` **except** `fonts.liquid` and `theme-styles-variables.liquid`
- `sections/*.liquid` **except** `product-information.liquid`
- `blocks/*.liquid` **except** `buy-buttons.liquid`
- `layout/theme.liquid`
- `locales/`
- `config/settings_schema.json`

## ⚠️ Files that always need manual merge

| File | Customization |
|------|--------------|
| `blocks/buy-buttons.liquid` | Grind selector |
| `sections/product-information.liquid` | Custom background |
| `snippets/fonts.liquid` | ITC Garamond declaration |
| `snippets/theme-styles-variables.liquid` | Font CSS variables |
| `config/settings_data.json` | All store settings — never overwrite |
| `sections/header-group.json` | Header layout |
| `sections/footer-group.json` | Footer layout |
| `templates/*.json` | Page layouts |
