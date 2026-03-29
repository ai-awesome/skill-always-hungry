# Scouting Run 2026-03-29 — michaelzuo-blog (Run 2)

## Stats
- Repos scanned: 12
- Repos with new content: 5
- Candidates found: 3
- Candidates passed: 2
- Candidates applied: 2

## Applied

### Add JSON-LD Article structured data to blog posts
- Source: https://github.com/leerob/next-mdx-blog
- Files changed: src/app/post/[slug]/page.tsx
- Key insight: Inject `<script type="application/ld+json">` with BlogPosting schema into each post page, using existing metadata (title, date, spoiler, author). Enables rich snippets in Google search results.

### Replace postbuild sitemap script with native Next.js sitemap.ts
- Source: https://github.com/leerob/next-mdx-blog
- Files changed: src/app/sitemap.ts (new), package.json, scripts/generate-sitemap.mjs (removed)
- Key insight: Next.js built-in `sitemap()` function generates sitemap.xml at build time with `export const dynamic = "force-static"` for static export compatibility. Type-safe, no postbuild script needed.

## Dropped

### Add robots.ts for proper robots.txt generation
- Source: https://github.com/nelsonlaidev/nelsonlai.dev
- Reason: public/robots.txt already exists with correct content

## Skipped Repos
- giscus/giscus — commenting system, valuable but requires GitHub Discussions setup
- ibelick/nim — Motion-Primitives animations, too different an architecture
- nelsonlaidev/nelsonlai.dev — comprehensive blog; robots.ts candidate dropped (already exists)
- ArtemKutsan/astro-citrus — Astro-based, different framework; Satori OG images require new deps
