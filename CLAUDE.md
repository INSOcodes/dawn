# CLAUDE.md - Dawn Theme Development Guide

This document provides guidance for AI assistants working with the Shopify Dawn theme codebase.

## Project Overview

**Dawn** is Shopify's official reference theme for Online Store 2.0. It follows a "HTML-first, JavaScript-only-as-needed" approach emphasizing performance, accessibility, and progressive enhancement.

**Version:** 8.0.0-rc
**Theme Type:** Shopify Online Store 2.0 Theme
**Primary Technologies:** Liquid, CSS, Vanilla JavaScript

## Core Philosophy

Dawn adheres to these principles - understand them before making changes:

1. **Web-native in its purest form** - No frameworks, libraries, or dependencies. Use native browser APIs.
2. **Lean, fast, and reliable** - Zero CLS, no render-blocking JS, no long tasks, no DOM manipulation before user input.
3. **JavaScript not required** - JS is only for progressive enhancement. Core functionality must work without JS.
4. **Server-rendered** - HTML is rendered by Shopify servers using Liquid. Business logic stays server-side.
5. **Functional, not pixel-perfect** - Progressive enhancement over polyfills; semantic markup over visual precision.

## Directory Structure

```
dawn/
├── assets/           # CSS and JavaScript files
│   ├── base.css      # Core styles and CSS custom properties
│   ├── global.js     # Core JavaScript and Web Components
│   ├── pubsub.js     # Simple pub/sub event system
│   └── component-*.css  # Component-specific styles
├── config/
│   ├── settings_schema.json  # Theme settings schema (editor UI)
│   └── settings_data.json    # Default theme settings values
├── layout/
│   ├── theme.liquid   # Main layout template
│   └── password.liquid # Password page layout
├── locales/          # Translation files (en.default.json is the source)
│   ├── en.default.json         # English translations
│   └── *.schema.json           # Theme editor translations
├── sections/         # Reusable page sections
│   ├── header.liquid           # Site header
│   ├── footer.liquid           # Site footer
│   ├── main-*.liquid           # Main content sections
│   └── *.json                  # Section groups
├── snippets/         # Reusable Liquid components
│   ├── card-product.liquid     # Product card component
│   ├── icon-*.liquid           # SVG icons
│   └── facets.liquid           # Collection filters
├── templates/        # Page templates (JSON format for OS 2.0)
│   ├── index.json              # Homepage
│   ├── product.json            # Product page
│   ├── collection.json         # Collection page
│   └── customers/              # Customer account templates
└── .theme-check.yml  # Theme Check linting configuration
```

## Development Commands

```bash
# Install Shopify CLI (required)
npm install -g @shopify/cli @shopify/theme

# Start development server
shopify theme dev

# Run linting (Theme Check)
shopify theme check

# Push theme to store
shopify theme push

# Pull theme from store
shopify theme pull
```

## Key Conventions

### Liquid Templates

- **Sections** (`sections/*.liquid`): Self-contained, reusable modules with their own settings schema
- **Snippets** (`snippets/*.liquid`): Reusable components without settings; document parameters in comments
- **Templates** (`templates/*.json`): JSON files defining section order and default settings

Section files must include a `{% schema %}` block at the end defining:
- `name`: Display name in theme editor
- `settings`: Configurable options
- `blocks`: Optional nested content blocks
- `presets`: Optional default configurations

### CSS Patterns

1. **CSS Custom Properties**: Defined in `theme.liquid` and `base.css`
   ```css
   --color-foreground: var(--color-base-text);
   --font-body-family: {{ settings.type_body_font.family }};
   ```

2. **Component CSS**: Each component has its own CSS file (e.g., `component-card.css`)

3. **Color Schemes**: Use `.color-background-1`, `.color-accent-1`, etc. classes

4. **Responsive**: Mobile-first with breakpoint at 750px and 990px
   ```css
   @media screen and (min-width: 750px) { }
   @media screen and (min-width: 990px) { }
   ```

5. **CSS Loading**: Non-critical CSS uses media="print" with onload swap:
   ```liquid
   <link rel="stylesheet" href="{{ 'component.css' | asset_url }}" media="print" onload="this.media='all'">
   ```

### JavaScript Patterns

1. **Web Components**: Custom elements extend HTMLElement
   ```javascript
   class QuantityInput extends HTMLElement {
     constructor() {
       super();
       // Initialize
     }
     connectedCallback() { }
     disconnectedCallback() { }
   }
   customElements.define('quantity-input', QuantityInput);
   ```

2. **Pub/Sub System** (`pubsub.js`): For cross-component communication
   ```javascript
   subscribe(PUB_SUB_EVENTS.cartUpdate, callback);
   publish(PUB_SUB_EVENTS.cartUpdate, data);
   ```

3. **Deferred Loading**: All JS uses `defer="defer"` attribute

4. **Progressive Enhancement**: Features work without JS; JS enhances the experience

5. **Event Constants** (`constants.js`): Define event names centrally

### Accessibility Requirements

- All interactive elements must be keyboard accessible
- Use proper ARIA attributes (`aria-expanded`, `aria-controls`, `aria-label`)
- Focus trapping for modals/drawers (see `trapFocus()` in `global.js`)
- `prefers-reduced-motion` media query support
- Skip links for keyboard navigation

### Translation Keys

Use translation keys instead of hardcoded strings:
```liquid
{{ 'products.product.add_to_cart' | t }}
```

Translations are in `locales/en.default.json` for storefront and `locales/en.default.schema.json` for theme editor.

## Important Files

| File | Purpose |
|------|---------|
| `layout/theme.liquid` | Main layout wrapper with CSS variables and global scripts |
| `assets/global.js` | Core Web Components (MenuDrawer, ModalDialog, SliderComponent, etc.) |
| `assets/pubsub.js` | Event pub/sub system |
| `assets/base.css` | Core CSS including color schemes and utility classes |
| `config/settings_schema.json` | Theme-wide settings for the editor |
| `snippets/card-product.liquid` | Product card with documented parameters |
| `sections/header.liquid` | Site header with navigation |
| `sections/main-product.liquid` | Product page main section |

## Quality Checks

Before submitting changes:

1. **Theme Check**: Run `shopify theme check` - all checks must pass
2. **No console errors**: Test in browser DevTools
3. **Keyboard navigation**: Test all interactive elements
4. **Performance**: No layout shifts, no render-blocking resources
5. **NoJS fallback**: Test with JavaScript disabled
6. **Responsive**: Test at 375px, 750px, 990px, and 1440px widths

## CI/CD

The repository uses GitHub Actions (`.github/workflows/ci.yml`):
- **Theme Check**: Validates Liquid code quality
- **Lighthouse CI**: Performance audits on home, product, and collection pages

## Common Tasks

### Adding a new section

1. Create `sections/your-section.liquid`
2. Add HTML structure with Liquid
3. Include `{% schema %}` block with settings
4. Create `assets/section-your-section.css` for styles
5. Add translations to `locales/en.default.json` and schema translations

### Adding a new snippet

1. Create `snippets/your-snippet.liquid`
2. Document accepted parameters in a `{% comment %}` block at the top
3. Use `{% render 'your-snippet', param: value %}` to include

### Modifying JavaScript

1. Edit the relevant file in `assets/`
2. For new components, define a Web Component extending HTMLElement
3. Register with `customElements.define('component-name', ComponentClass)`
4. Never add external dependencies

### Adding theme settings

1. Edit `config/settings_schema.json` to add setting definitions
2. Access in Liquid via `{{ settings.your_setting }}`
3. Add translation keys in `locales/*.schema.json`

## Anti-patterns to Avoid

- Adding external JavaScript libraries or frameworks
- Using JavaScript for functionality that can be done with HTML/CSS
- Render-blocking resources
- DOM manipulation on page load (only on user interaction)
- Inline styles (use CSS custom properties)
- Hardcoded strings (use translation keys)
- Breaking existing accessibility features
- Adding polyfills (use progressive enhancement instead)
