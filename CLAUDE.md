# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a shared SCSS styles library for the DigiDex project. It provides theming, typography, layout, and component styles used across multiple services (CMS, app, storefront).

## Build Commands

```bash
# Compile SCSS (from CMS backend - primary consumer)
cd ../cms/backend
python manage.py sass -g website/static/website/src/custom.scss website/static/website/css/

# Production build with compression
python manage.py sass -t compressed website/static/website/src/custom.scss website/static/website/css/
```

## Architecture

The library uses the Sass module system (`@use`/`@forward`) organized in layers:

```
main.scss                    # Entry point - imports all layers
├── abstracts/               # Variables, themes, mixins (no CSS output)
│   ├── _colors.scss         # Color palettes (cyber, nature themes)
│   ├── _themes.scss         # Theme maps and active theme selection
│   ├── _variables.scss      # CSS custom properties, typography scales
│   ├── _mixins.scss         # Responsive breakpoints, flex helpers
│   └── _normalize.scss      # Browser normalization
├── base/                    # Global styles, typography, resets
├── layout/                  # Header, footer, navigation, hero sections
├── pages/                   # Page-specific styles (accounts, blog, etc.)
└── components/              # UI components (buttons, forms)
```

## Theming

Themes are defined in `abstracts/_themes.scss` as Sass maps with color palettes:

**Current themes:**
- `g` (green - default) - Primary: #4CAF50, Secondary: #2196F3
- `b` (blue) - Primary: #2196F3, Secondary: #4CAF50

**To switch themes**, edit `abstracts/_themes.scss`:
```scss
$active-theme-name: 'g';  // Change to 'b' or add new theme
```

**CSS Custom Properties** are auto-generated at `:root`:
```css
:root {
  --color-primary: #4CAF50;
  --color-secondary: #2196F3;
  --color-accent: #FF9800;
  /* ... more colors ... */
}
```

Use in CSS/SCSS:
```scss
.button {
  background: var(--color-primary);
  color: white;
}
```

**Adding a new theme:**
1. Add to `$themes` map in `_themes.scss`:
   ```scss
   'dark': (
     'primary': #BB86FC,
     'secondary': #03DAC6,
     // ... other colors
   )
   ```
2. Change `$active-theme-name: 'dark'`
3. Recompile: `python manage.py sass -g custom.scss output.css`

## Import Pattern

All modules expose content via `_index.scss` files using `@forward`. This allows clean imports:

**Import all abstracts** (variables, themes, mixins available without namespace):
```scss
@use "abstracts" as *;

.element {
  color: $text-primary;           // Variable from abstracts
  padding: map-get($active-theme, 'primary');  // Access theme
  @include flex-center;           // Mixin from abstracts
}
```

**Import specific module with namespace**:
```scss
@use "abstracts/colors" as colors;
@use "base";

.element {
  background: colors.$primary;    // Namespaced access
  color: base.$text-color;        // From base layer
}
```

**Import compiled CSS output**:
```scss
// For consumers that don't process SCSS
@import "css/main.css";  // Compiled output
```

**Full layer imports** (for building on top of all styles):
```scss
@use "main.scss" as *;  // Brings in all layers: abstracts, base, layout, components

.custom-element {
  @extend %clearfix;    // Use utilities from base
  @include breakpoint-md { /* responsive */ }  // Use mixins from abstracts
}
```

## Floating Labels (CSS-Only)

A pure CSS floating label implementation using the `:not(:placeholder-shown)` selector. No JavaScript required.

**Files:**
- `components/forms/TextField.scss` - Input field styles + floating label CSS
- `components/forms/_index.scss` - Module export

**How It Works:**

The label floats based on CSS pseudo-selectors:
- **At rest** (no value): Label positioned inside input with transparent background
- **On focus OR input has value**: Label floats above input border with background color to create visual gap

```scss
// Base state - label inside input
.text-field + .form-label {
  position: absolute;
  top: 1.25em;
  background-color: transparent;  // No visible background
  transition: top 0.2s ease, font-size 0.2s ease, color 0.2s ease, background-color 0.2s ease;
}

// Floating state - label above input
.text-field:focus + .form-label,
.text-field:not(:placeholder-shown) + .form-label {
  top: 0.3em;
  font-size: 0.75rem;
  background-color: var(--foreground);  // Covers border line
  color: var(--accent-a1);
}
```

**HTML Structure:**

```html
<div class="field-group">
  <input class="text-field" type="text" placeholder="Label Text" />
  <label class="form-label">Label Text</label>
</div>
```

**Key Points:**
- `.field-group` must be `position: relative` to anchor absolute-positioned label
- Placeholder and label should have the same text (placeholder is invisible, label shows)
- Label background matches input background to hide border line when floating
- Works in all modern browsers (`:not(:placeholder-shown)` supported in Chrome 105+, Firefox 78+, Safari 15.4+)
- No class manipulation or JavaScript events needed

**Variant: Transparent Input**

For inputs with transparent backgrounds (e.g., dark theme):

```scss
.text-field.transparent + .form-label {
  background-color: transparent;  // No background for transparent variants
}

.text-field.transparent:focus + .form-label,
.text-field.transparent:not(:placeholder-shown) + .form-label {
  color: var(--foreground);
}
```

**Used By:**
- Account forms in ID service (`id/frontend/src/components/forms/`)
- App service forms (`app/frontend/src/components/forms/`)
