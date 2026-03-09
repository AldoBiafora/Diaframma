# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Diaframma** is an all-in-one platform for photographers to deliver work in a premium experience. Every text and image on the site is stored in the database and editable from an admin panel — no hardcoded content in the frontend.

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 15 (App Router, React Server Components, TypeScript) |
| Styling | Vanilla CSS with custom variables (no CSS frameworks) |
| Backend / DB | Supabase (PostgreSQL, Auth, Storage, Row Level Security) |
| Images | Supabase Storage + `next/image` |
| Deploy | Vercel |

## Commands

```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # TypeScript check (tsc --noEmit)
```

## Architecture

### Content-Driven Principle

The frontend contains **no hardcoded text or image URLs**. Everything is read from the `content_sections` table. The admin modifies any label or image, and the site updates automatically.

```
Admin Panel → edits text/images → content_sections (DB)
                                        ↓
                                Public Site (reads from DB)

Admin Panel → upload media → Supabase Storage → URL saved in DB
```

### Key Helper

`src/lib/content.ts` exposes `getContent(key)` and `getContentByPage(page)` — these are the primary functions for fetching site content. Always use these helpers rather than querying `content_sections` directly from page components.

### Database Tables

- **`site_settings`** — global config: colors, fonts, social links (single row)
- **`content_sections`** — all site texts and images, keyed by `section_key` (e.g. `hero_title`, `about_image`), grouped by `page` (`home`, `gallery`, `global`)
- **`folders`** — photo albums/events with slug, cover image, visibility toggle
- **`photos`** — individual photos with FK to folder, storage path, dimensions

### RLS Rules

- Public read on all tables (`folders` filtered by `is_visible = true`)
- Write (INSERT, UPDATE, DELETE) only for authenticated admin user

### Supabase Clients

`src/lib/supabase/` contains three separate clients:
- **browser client** — for client components
- **server client** — for React Server Components and Server Actions
- **middleware** — for auth protection of `/admin/*` routes

Never use the browser client in Server Components or vice versa.

### Route Structure

```
/                          — Homepage (reads 'home' sections from DB)
/gallery                   — Grid of visible folders
/gallery/[slug]            — Masonry photo grid + fullscreen lightbox
/admin/login               — Admin login
/admin                     — Dashboard
/admin/settings            — Site settings (colors, fonts, social)
/admin/content             — Edit all texts and images grouped by page
/admin/folders             — Folder CRUD with drag & drop reorder
/admin/folders/[slug]/photos — Photo management (multi-upload, reorder)
```

### Component Organization

- `src/components/ui/` — generic primitives (Button, Input, Toast, Modal)
- `src/components/admin/` — Sidebar, ContentEditor, ImageUploader
- `src/components/public/` — DynamicSection, MasonryGrid, Lightbox

### CSS Approach

Three stylesheets, no utility frameworks:
- `styles/globals.css` — reset + design system (CSS custom properties)
- `styles/admin.css` — admin panel styles
- `styles/public.css` — public site styles

Design tokens (colors, fonts) from `site_settings` are injected as CSS custom properties at the root layout level.

## Development Phases

The project follows 10 phases defined in `DIAFRAMMA_PROJECT_PLAN.md`:
1. Infrastructure & Setup
2. Database (tables, RLS, seed)
3. Admin Authentication
4. Admin: Settings page
5. Admin: Content management
6. Admin: Folder management
7. Admin: Photo management
8. Public: Layout & Homepage
9. Public: Gallery & lightbox
10. SEO, Performance & Deploy
