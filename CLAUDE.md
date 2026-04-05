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
