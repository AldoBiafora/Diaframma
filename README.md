# Diaframma

> Passare dal "caos dei file" all'eleganza professionale.

**Diaframma** è una piattaforma all-in-one per fotografi che vogliono trasformare la consegna dei lavori in un'esperienza premium. Ogni testo e ogni immagine del sito è salvato nel database e modificabile dall'admin panel — nessun contenuto hardcoded nel frontend.

---

## Stack

| Layer | Tecnologia |
|-------|-----------|
| Frontend | Next.js 15 (App Router, React Server Components, TypeScript) |
| Styling | Vanilla CSS con design system a variabili custom |
| Backend / DB | Supabase (PostgreSQL, Auth, Storage, Row Level Security) |
| Immagini | Supabase Storage + `next/image` |
| Deploy | Vercel |

---

## Avvio rapido

```bash
# 1. Installa le dipendenze
npm install

# 2. Crea il file delle variabili d'ambiente
cp .env.local.example .env.local
# → Compila con le tue credenziali Supabase

# 3. Avvia il dev server
npm run dev
```

### Variabili d'ambiente richieste

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key   # solo server-side
```

### Comandi disponibili

```bash
npm run dev          # Dev server con hot reload
npm run build        # Build di produzione
npm run lint         # ESLint
npm run type-check   # TypeScript check (tsc --noEmit)
```

---

## Architettura

### Principio content-driven

Il frontend non contiene testi o URL di immagini hardcoded. Tutto è letto dalla tabella `content_sections`. Il fotografo modifica qualsiasi contenuto dall'admin e il sito si aggiorna automaticamente.

```
Admin Panel → modifica testi/immagini → content_sections (DB)
                                               ↓
                                         Sito Pubblico

Admin Panel → upload media → Supabase Storage → URL salvato in DB
```

L'helper principale è `src/lib/content.ts` che espone `getContent(key)` e `getContentByPage(page)`. Usare sempre questi helper invece di interrogare `content_sections` direttamente dai componenti.

### Struttura del progetto

```
src/
├── app/
│   ├── layout.tsx                      — Root layout (legge site_settings + contenuti globali)
│   ├── page.tsx                        — Homepage
│   ├── gallery/
│   │   ├── page.tsx                    — Griglia cartelle visibili
│   │   └── [slug]/page.tsx             — Foto della cartella + lightbox
│   └── admin/
│       ├── login/page.tsx              — Login admin
│       ├── layout.tsx                  — Layout admin protetto (sidebar)
│       ├── page.tsx                    — Dashboard
│       ├── settings/page.tsx           — Impostazioni sito
│       ├── content/page.tsx            — Gestione testi e immagini
│       └── folders/
│           ├── page.tsx                — Lista cartelle
│           └── [slug]/photos/page.tsx  — Gestione foto
├── components/
│   ├── ui/                             — Button, Input, Toast, Modal
│   ├── admin/                          — Sidebar, ContentEditor, ImageUploader
│   └── public/                         — DynamicSection, MasonryGrid, Lightbox
├── lib/
│   ├── supabase/                       — Client browser, server e middleware
│   ├── content.ts                      — getContent(), getContentByPage()
│   └── utils.ts
└── styles/
    ├── globals.css                     — Reset + design system
    ├── admin.css                       — Stili admin panel
    └── public.css                      — Stili sito pubblico
```

---

## Database

### Tabelle

**`site_settings`** — Configurazione globale (riga singola)

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `site_title` | TEXT | Titolo del sito |
| `primary_color` | TEXT | Colore primario (es. `#1a1a1a`) |
| `accent_color` | TEXT | Colore accento (es. `#d4a853`) |
| `font_heading` | TEXT | Font titoli |
| `font_body` | TEXT | Font corpo |
| `instagram_url` | TEXT | Link Instagram |
| `email` | TEXT | Email di contatto |

**`content_sections`** — Tutti i testi e le immagini del sito

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `section_key` | TEXT | Chiave univoca (es. `hero_title`) |
| `page` | TEXT | Pagina: `home`, `gallery`, `global` |
| `content_type` | TEXT | Tipo: `text`, `image`, `rich_text` |
| `text_value` | TEXT | Valore testuale |
| `image_url` | TEXT | URL immagine da Storage |
| `display_label` | TEXT | Nome leggibile in admin |

**`folders`** — Album/eventi fotografici

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `name` | TEXT | Nome della cartella |
| `slug` | TEXT | Slug per URL |
| `cover_image_url` | TEXT | Immagine di copertina |
| `event_date` | DATE | Data evento |
| `is_visible` | BOOLEAN | Visibile sul sito pubblico |

**`photos`** — Singole foto

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `folder_id` | UUID | FK → folders |
| `storage_path` | TEXT | Path in Supabase Storage |
| `url` | TEXT | URL pubblico |
| `width` / `height` | INT | Dimensioni in px |
| `sort_order` | INT | Ordine nella cartella |

### Sicurezza (RLS)

- Lettura pubblica su tutte le tabelle (`folders` filtrata per `is_visible = true`)
- Scrittura (INSERT, UPDATE, DELETE) solo per utente admin autenticato

### Client Supabase

`src/lib/supabase/` contiene tre client separati — usare quello corretto per contesto:

| File | Dove si usa |
|------|-------------|
| `browser.ts` | Client Components |
| `server.ts` | Server Components e Server Actions |
| `middleware.ts` | Middleware Next.js per protezione route |

---

## Admin Panel

Accessibile su `/admin` (protetto da autenticazione).

| Sezione | Route | Funzionalità |
|---------|-------|-------------|
| Dashboard | `/admin` | Panoramica |
| Impostazioni | `/admin/settings` | Colori, font, link social |
| Contenuti | `/admin/content` | Editing di tutti i testi e immagini, raggruppati per pagina |
| Cartelle | `/admin/folders` | CRUD cartelle, drag & drop riordino, toggle visibilità |
| Foto | `/admin/folders/[slug]/photos` | Upload multiplo, riordino, sostituzione, eliminazione |

---

## Sito Pubblico

| Route | Descrizione |
|-------|-------------|
| `/` | Homepage: hero, about, cartelle in evidenza, CTA |
| `/gallery` | Griglia di tutte le cartelle visibili |
| `/gallery/[slug]` | Masonry grid + lightbox fullscreen con swipe |

---

## Fasi di sviluppo

| # | Fase | Stato |
|---|------|-------|
| 1 | Infrastruttura & Setup | ✅ |
| 2 | Database (tabelle, RLS, seed) | 🔲 |
| 3 | Autenticazione Admin | 🔲 |
| 4 | Admin: Impostazioni | 🔲 |
| 5 | Admin: Gestione Contenuti | 🔲 |
| 6 | Admin: Gestione Cartelle | 🔲 |
| 7 | Admin: Gestione Foto | 🔲 |
| 8 | Sito Pubblico: Layout & Homepage | 🔲 |
| 9 | Sito Pubblico: Galleria & Lightbox | 🔲 |
| 10 | SEO, Performance & Deploy | 🔲 |
