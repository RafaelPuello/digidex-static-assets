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

Themes are defined in `abstracts/_themes.scss` as Sass maps. The active theme is set via `$active-theme-name` and CSS custom properties are generated at `:root`.

Current themes: `g` (green - default), `b` (blue)

To switch themes, change `$active-theme-name` in `_themes.scss`.

## Import Pattern

All modules expose content via `_index.scss` files using `@forward`. Consumer files should use:

```scss
@use "abstracts" as *;  // Makes all abstract members available
@use "base";            // Namespaced imports for other layers
```
