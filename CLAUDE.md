# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fuwari is a static blog template built with Astro and Tailwind CSS. The site is configured to deploy at https://laplusda.com/ with traditional Chinese (zh_TW) as the default language.

## Development Commands

```bash
# Install dependencies (MUST use pnpm)
pnpm install

# Start development server at localhost:4321
pnpm dev

# Build for production with search functionality
pnpm build

# Preview production build
pnpm preview

# Create a new blog post
pnpm new-post <filename>

# Format code using Biome
pnpm format

# Lint and fix code
pnpm lint

# Type check TypeScript
pnpm type-check

# Check for Astro errors
pnpm check
```

## Architecture

### Content Management
- **Blog posts**: Located in `/src/content/posts/`
- **Post format**: Markdown with frontmatter (title, published, description, image, tags, category, draft)
- **Markdown syntax examples**: Reference `/src/content/posts/markdown.md` and `/src/content/posts/markdown-extended.md`
- **Categories**: Must use predefined categories from `.cursor/rules/posts-rule.mdc`
- **Tags**: Follow the naming conventions in the posts rule file

### Key Configuration Files
- `src/config.ts`: Site configuration (title, theme, profile, navigation)
- `astro.config.mjs`: Astro build configuration and integrations
- `src/content/config.ts`: Content collection schemas

### Styling & Theming
- Uses Tailwind CSS with custom color variables
- Theme color system based on CSS custom properties
- Dark mode support with configurable hue values
- Custom Expressive Code configuration for code blocks

### Markdown Extensions
- GitHub-style admonitions (note, tip, warning, caution, important)
- GitHub repository cards
- Math support via KaTeX
- Enhanced code blocks with Expressive Code
- Automatic table of contents generation

## Writing Guidelines

When creating or editing blog posts:

1. **Frontmatter Requirements**:
   - Use valid categories from: 前端, 後端, AI, 工具, 個人專案, 活動, 行銷, 網站追蹤, 伺服器, 個人筆記
   - Keep tags to 3-5 per post
   - Published date format: YYYY-MM-DD

2. **Reference Format**:
   ```markdown
   > 參考資料：
   >
   > [Link Title](URL)
   >
   > [Another Link](URL)
   ```

3. **File Organization**:
   - Single posts: `/src/content/posts/post-name.md`
   - Posts with assets: `/src/content/posts/post-name/index.md` with related files in same folder

## Important Rules

Refer to `.cursor/rules/posts-rule.mdc` for detailed guidelines on:
- Post structure and metadata
- Category and tag systems
- SEO optimization
- Content quality standards