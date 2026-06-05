# AGENTS.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs_new/AGENTS.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: name: sglang-docs-mintlify description: Build and maintain the SGLang documentation site and integrated cookbook using Mintlify. Use when creating docs pages, configuring navigation, adding components, or setting up API references. / 该文档围绕 SGLang Mintlify documentation guide for agents 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** name: sglang-docs-mintlify description: Build and maintain the SGLang documentation site and integrated cookbook using Mintlify. Use when creating docs pages, configuring navigation, adding components, or setting up API references.
**CN:** 本节围绕 Overview 展开，概述了 API, Build, Git-based, Apache-2.0 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Non-negotiables
**EN:** **Prefer fixing the docs-site version of an internal link** instead of copying links from older docs. **Keep examples copy/pasteable.** Use placeholders consistently (e.g., MODEL_PATH, HF_TOKEN, HOST, PORT).
**CN:** 本节围绕 Non-negotiables 展开，概述了 HOST, PORT, HF_TOKEN, MODEL_PATH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Source of truth hierarchy
**EN:** **This repo** - docs.json for site structure + navigation - existing MDX pages for voice + conventions 2. **Canonical current docs** - docs.sglang.io (Sphinx site) is currently the reference structure and content baseline.
**CN:** 本节围绕 Source of truth hierarchy 展开，概述了 MDX, site, Sphinx, Cookbook 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Voice and structure
**EN:** * Second person (“you”), active voice. * Prefer **short, scannable sections** with clear outcomes.
**CN:** 本节围绕 Voice and structure 展开，概述了 Put, Second, Prefer, Headings 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Technical accuracy patterns
**EN:** This content focuses on Technical accuracy patterns and highlights HF_TOKEN, CPU, etc, --tp.
**CN:** 本节围绕 Technical accuracy patterns 展开，概述了 HF_TOKEN, CPU, etc, --tp 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Mintlify best practices
**EN:** **Always consult mintlify.com/docs for components, configuration, and latest features.** If you are not already connected to the Mintlify MCP server, https://mintlify.com/docs/mcp, add it so that you can search more efficiently.
**CN:** 本节围绕 Mintlify best practices 展开，概述了 Mintlify, MDX, Always, components 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Understand the project
**EN:** Read docs.json in the project root. This file defines the entire site: navigation structure, theme, colors, links, API and specs.
**CN:** 本节围绕 Understand the project 展开，概述了 site, navigation, API, Read 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Check for existing content
**EN:** Search the docs before creating new pages. You may need to: * Update an existing page instead of creating a new one * Add a section to an existing page * Link to existing content rather than duplicating
**CN:** 本节围绕 Check for existing content 展开，概述了 existing, Add, new, Link 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Read surrounding content
**EN:** Before writing, read 2-3 similar pages to understand the site's voice, structure, formatting conventions, and level of detail.
**CN:** 本节围绕 Read surrounding content 展开，概述了 Before, read, site, voice 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Understand Mintlify components
**EN:** Review the Mintlify components to select and use any relevant components for the documentation request that you are working on.
**CN:** 本节围绕 Understand Mintlify components 展开，概述了 Review, Mintlify, components, select 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CLI commands
**EN:** * npm i -g mint - Install the Mintlify CLI * mint dev - Local preview at localhost:3000 * mint broken-links - Check internal links * mint a11y - Check for accessibility issues in content * mint rename - Rename/move files and update references * mint validate - Validate documentation builds
**CN:** 本节围绕 CLI commands 展开，概述了 mint, Check, Local, Install 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Required files
**EN:** * docs.json - Site configuration (navigation, theme, integrations, etc.). See global settings for all options.
**CN:** 本节围绕 Required files 展开，概述了 See, Site, docs.json, etc 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example file structure
**EN:** `` project/ ├── docs.json # Site configuration ├── introduction.mdx ├── quickstart.mdx ├── guides/ │ └── example.mdx ├── openapi.yml # API specification ├── images/ # Static assets │ └── example.png └── snippets/ # Reusable components └── component.jsx ``
**CN:** 本节围绕 Example file structure 展开，概述了 API, Site, Static, Reusable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Page frontmatter
**EN:** Every page requires title in its frontmatter. Include description for SEO and navigation.
**CN:** 本节围绕 Page frontmatter 展开，概述了 SEO, title, mode, custom 等要点，并说明相关配置、流程、示例或限制条件。

### Section: File conventions
**EN:** This content focuses on File conventions and highlights URLs, Match, paths, naming.
**CN:** 本节围绕 File conventions 展开，概述了 URLs, Match, paths, naming 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Organize content
**EN:** When a user asks about anything related to site-wide configurations, start by understanding the global settings. See if a setting in the docs.json file can be updated to achieve what the user wants.
**CN:** 本节围绕 Organize content 展开，概述了 See, user, docs.json, asks 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Navigation
**EN:** This section provides a comparison table for Navigation, covering columns such as Pattern, When to use and examples such as **Groups**, **Tabs**, **Anchors**, **Dropdowns**.
**CN:** 本节围绕 Navigation 展开，概述了 API, groups, Add, Tabs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Links and paths
**EN:** * **Internal links:** Root-relative, no extension: /getting-started/quickstart * **Images:** Store in /images, reference as /images/example.png * **External links:** Use full URLs, they open in new tabs automatically
**CN:** 本节围绕 Links and paths 展开，概述了 URLs, Store, links, Images 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Customize docs sites
**EN:** **What to customize where:** * **Brand colors, fonts, logo** → docs.json. See global settings * **Component styling, layout tweaks** → custom.css at project root * **Dark mode** → Enabled by default.
**CN:** 本节围绕 Customize docs sites 展开，概述了 docs.json, custom.css, See, Dark 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Components
**EN:** This section provides a comparison table for Components, covering columns such as Need, Use and examples such as Hide optional details, Long code examples, User chooses one option, Linked navigation cards.
**CN:** 本节围绕 Components 展开，概述了 API, Tip, APIs, Need 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reusable content
**EN:** **When to use snippets:** * Exact content appears on more than one page * Complex components you want to maintain in one place * Shared content across teams/repos **When NOT to use snippets:** * Slight variations needed per page (leads to complex props) Import snippets with import from "/path/to/snippet-name.jsx".
**CN:** 本节围绕 Reusable content 展开，概述了 snippets, one, Exact, Shared 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Voice and structure
**EN:** This content focuses on Voice and structure and highlights Sentence, Getting, Expandable, Lead.
**CN:** 本节围绕 Voice and structure 展开，概述了 Sentence, Getting, Expandable, Lead 等要点，并说明相关配置、流程、示例或限制条件。

### Section: What to avoid
**EN:** This content focuses on What to avoid and highlights just, Never, Watch, Filler.
**CN:** 本节围绕 What to avoid 展开，概述了 just, Never, Watch, Filler 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Formatting
**EN:** * All code blocks must have language tags * All images and media must have descriptive alt text * Use bold and italics only when they serve the reader's understanding--never use text styling just for decoration * No decorative formatting or emoji
**CN:** 本节围绕 Formatting 展开，概述了 must, text, alt, code 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Code examples
**EN:** * Keep examples simple and practical * Use realistic values (not "foo" or "bar") * One clear example is better than multiple variations * Test that code works before including it
**CN:** 本节围绕 Code examples 展开，概述了 One, Keep, Test, foo 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deploy
**EN:** Mintlify deploys automatically when changes are pushed to the connected Git repository.
**CN:** 本节围绕 Deploy 展开，概述了 Git, Add, SEO, DNS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 1. Understand the task
**EN:** Identify what needs to be documented, which pages are affected, and what the reader should accomplish afterward. If any of these are unclear, ask.
**CN:** 本节围绕 1. Understand the task 展开，概述了 Identify, ask, needs, these 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2. Research
**EN:** * Read docs.json to understand the site structure * Search existing docs for related content * Read similar pages to match the site's style
**CN:** 本节围绕 2. Research 展开，概述了 Read, site, Search, docs.json 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 3. Plan
**EN:** * Synthesize what the reader should accomplish after reading the docs and the current content * Propose any updates or new content * Verify that your proposed changes will help readers be successful
**CN:** 本节围绕 3. Plan 展开，概述了 Verify, Propose, content, Synthesize 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 4. Write
**EN:** * Start with the most important information * Keep sections focused and scannable * Use components appropriately (don't overuse them) * Mark anything uncertain with a TODO comment: ``mdx theme= ``
**CN:** 本节围绕 4. Write 展开，概述了 TODO, Keep, Mark, Start 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 5. Update navigation
**EN:** If you created a new page, add it to the appropriate group in docs.json.
**CN:** 本节围绕 5. Update navigation 展开，概述了 docs.json, new, add, group 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 6. Verify
**EN:** This content focuses on 6. Verify and highlights Run, New, mint, TODOs.
**CN:** 本节围绕 6. Verify 展开，概述了 Run, New, mint, TODOs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Migrations
**EN:** If a user asks about migrating to Mintlify, ask if they are using ReadMe or Docusaurus. If they are, use the @mintlify/scraping CLI to migrate content.
**CN:** 本节围绕 Migrations 展开，概述了 Mintlify, CLI, MDX, ReadMe 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hidden pages
**EN:** Any page that is not included in the docs.json navigation is hidden. Use hidden pages for content that should be accessible by URL or indexed for the assistant or search, but not discoverable through the sidebar navigation.
**CN:** 本节围绕 Hidden pages 展开，概述了 URL, hidden, docs.json, navigation 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Exclude pages
**EN:** The .mintignore file is used to exclude files from a documentation repository from being processed.
**CN:** 本节围绕 Exclude pages 展开，概述了 mintignore, files, being, exclude 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common gotchas
**EN:** **Component imports** - JSX components need explicit import, MDX components don't 2. **Frontmatter required** - Every MDX file needs title at minimum 3.
**CN:** 本节围绕 Common gotchas 展开，概述了 mint.json, MDX, JSX, Code 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Resources
**EN:** * Documentation * https://github.com/sgl-project/sglang * Configuration schema * Feature requests * Bugs and feedback
**CN:** 本节围绕 Resources 展开，概述了 Bugs, Feature, Configuration, schema 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** docs.json / **CN:** docs.json
- **EN:** content / **CN:** content
- **EN:** Mintlify / **CN:** Mintlify
- **EN:** navigation / **CN:** navigation
- **EN:** API / **CN:** API
- **EN:** components / **CN:** components
- **EN:** MDX / **CN:** MDX
- **EN:** site / **CN:** site

## Dependencies / 依赖关系
- `openapi.yml`
