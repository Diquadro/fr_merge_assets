# CSS/JS Bundling Feature Request for html-bundler-webpack-plugin

## Overview
This repository demonstrates a feature request for **automatic bundling of CSS and JavaScript assets** in html-bundler-webpack-plugin. Currently, the plugin extracts each script/stylesheet as separate files, but we need them bundled per page entry point.

## Current Behavior vs Expected

### Current Output 
- **12 separate files**: 6 JS files + 6 CSS files
- Each `<script>` and `<link>` in Pug templates becomes a separate asset
- HTML contains multiple `<script>` and `<link>` tags

```html
<!-- Current: Multiple separate files -->
<link href="css/base.e5548910.css" rel="stylesheet">
<link href="css/index.8f2d284e.css" rel="stylesheet">
<link href="css/p1.8bbfdf28.css" rel="stylesheet">
<!-- ... 3 more CSS files -->

<script src="js/base.d89dccf4.js"></script>
<script src="js/index.fe495907.js"></script>
<!-- ... 4 more JS files -->
```

### Expected Output 
- **2 bundled files per page**: 1 CSS bundle + 1 JS bundle
- All dependencies automatically combined into page-level bundles
- Clean HTML with single tags per asset type

```html
<!-- Expected: Single bundled files per page -->
<link href="css/index.bundle.########.css" rel="stylesheet" />
<script src="js/index.bundle.########.js" defer></script>
```

## Project Structure

```
src/
├── layouts/base/          # Base layout (used by all pages)
│   ├── base.pug
│   ├── base.scss
│   └── base.ts
├── pages/index/           # Index page
│   ├── index.pug         # Extends base, includes partials
│   ├── index.scss
│   └── index.ts
└── partials/              # Reusable components
    ├── p1/ p2/ p3/ p4/   # Various partials with their own assets
```

## Dependency Chain

The bundling should follow the natural Pug dependency chain:

```
index.pug (entry point)
├── extends base.pug          → Include base.scss + base.ts
├── includes p1.pug           → Include p1.scss + p1.ts  
├── includes p2.pug           → Include p2.scss + p2.ts
│   └── includes p4.pug       → Include p4.scss + p4.ts
└── includes p3.pug           → Include p3.scss + p3.ts
    └── includes p4.pug       → Include p4.scss + p4.ts (deduplicated)
```

**Result**: Single `index.bundle.css` with all styles, single `index.bundle.js` with all scripts.

## Test Files

- **`build/`** - Current plugin output (separate files)
- **`expected/`** - Desired bundled output  
- **`dist/`** - Different project for comparison
- **`webpack.config.js`** - Simple, standard configuration

## Use Case

This is essential for:
- **Performance**: Fewer HTTP requests
- **Maintainability**: Automatic dependency resolution
- **Modern web practices**: Bundled assets per page
- **Large projects**: With many shared partials and layouts

## Configuration

Current webpack config is standard - no special bundling options needed:

```js
new HtmlBundlerPlugin({
  entry: {
    index: 'src/pages/index/index.pug',
  },
  js: {
    filename: 'js/[name].[contenthash:8].js',
  },
  css: {
    filename: 'css/[name].[contenthash:8].css',
  },
  preprocessor: 'pug',
})
```

## Expected Feature

The plugin should automatically:
1. **Trace dependencies** from Pug entry points (extends/includes)
2. **Collect all scripts/styles** from the dependency chain
3. **Bundle them per entry point** (not extract separately)
4. **Deduplicate shared dependencies** (like p4.pug included multiple times)
5. **Generate single bundles** named after the entry point

## Repository Purpose

This repo serves as a **minimal reproducible example** demonstrating:
- ✅ Current separate file extraction
- ✅ Expected bundled output
- ✅ Realistic project structure with layouts/partials
- ✅ Both TypeScript and SCSS assets
- ✅ Template inheritance and inclusion patterns

Ready for testing implementation of the bundling feature.
