# DIAFRAMMA

> Passare dal "caos dei file" all'eleganza professionale.

**Diaframma** è la piattaforma all-in-one dedicata ai fotografi che vogliono trasformare la consegna dei lavori in un'esperienza premium. Ogni testo e ogni immagine del sito è salvato a database e modificabile dall'admin panel, senza toccare il codice.

---

## Stack Tecnologico

| Layer | Tecnologia |
|-------|-----------|
| Frontend | Next.js 15 (App Router, React Server Components, TypeScript) |
| Styling | Vanilla CSS con variabili custom e design system proprietario |
| Backend / DB | Supabase (PostgreSQL, Auth, Storage, Row Level Security) |
| Immagini | Supabase Storage + `next/image` per ottimizzazione automatica |
| Deploy | Vercel |

---

## Architettura Content-Driven

### Principio chiave

Il frontend **non contiene testi o URL di immagini hardcoded**. Tutto è letto dal database tramite la tabella `content_sections`. Il fotografo modifica qualsiasi label o immagine dall'admin e il sito si aggiorna automaticamente.

### Flusso dati

```
Admin Panel → modifica testi/immagini → content_sections (DB)
                                              ↓
                                        Sito Pubblico (legge da DB)

Admin Panel → upload media → Supabase Storage → URL salvato in DB
```

---

## Database Schema

### `site_settings` — Impostazioni globali

Configurazione generale del sito: colori, font, link social.

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `id` | UUID | Primary key |
| `site_title` | TEXT | Titolo del sito (default: "Diaframma") |
| `primary_color` | TEXT | Colore primario (default: `#1a1a1a`) |
| `accent_color` | TEXT | Colore accento (default: `#d4a853`) |
| `font_heading` | TEXT | Font titoli (default: Playfair Display) |
| `font_body` | TEXT | Font corpo (default: Inter) |
| `instagram_url` | TEXT | Link Instagram |
| `email` | TEXT | Email di contatto |
| `website_url` | TEXT | Sito web esterno |

### `content_sections` — Tutti i testi e le immagini del sito

Ogni riga rappresenta un elemento del sito (titolo, sottotitolo, immagine, CTA, label di navigazione, ecc.).

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `id` | UUID | Primary key |
| `section_key` | TEXT | Chiave univoca (es. `hero_title`, `about_image`) |
| `page` | TEXT | Pagina di appartenenza (`home`, `gallery`, `global`) |
| `content_type` | TEXT | Tipo: `text`, `image`, `rich_text` |
| `text_value` | TEXT | Valore testuale |
| `image_url` | TEXT | URL immagine da Storage |
| `display_label` | TEXT | Nome leggibile per l'admin ("Titolo Hero") |
| `sort_order` | INT | Ordine di visualizzazione |

**Esempio contenuti:**

| section_key | page | tipo | valore | label admin |
|------------|------|------|--------|-------------|
| `hero_title` | home | text | "Catturare l'essenza" | Titolo Hero |
| `hero_subtitle` | home | text | "Fotografia d'autore" | Sottotitolo Hero |
| `hero_image` | home | image | /hero.jpg | Immagine Hero |
| `about_title` | home | text | "Chi Sono" | Titolo About |
| `about_text` | home | rich_text | "Sono un fotografo..." | Testo About |
| `about_image` | home | image | /about.jpg | Foto About |
| `logo_image` | global | image | /logo.png | Logo |
| `nav_home` | global | text | "Home" | Label Nav Home |
| `nav_gallery` | global | text | "Portfolio" | Label Nav Portfolio |
| `footer_text` | global | text | "© 2026 Diaframma" | Testo Footer |
| `cta_text` | home | text | "Scopri i miei lavori" | Testo CTA |

### `folders` — Cartelle / Eventi

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `id` | UUID | Primary key |
| `name` | TEXT | Nome della cartella |
| `slug` | TEXT | Slug univoco per URL |
| `description` | TEXT | Descrizione evento |
| `cover_image_url` | TEXT | Immagine di copertina |
| `event_date` | DATE | Data dell'evento |
| `sort_order` | INT | Ordine visualizzazione |
| `is_visible` | BOOLEAN | Visibile sul sito pubblico |

### `photos` — Foto

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `id` | UUID | Primary key |
| `folder_id` | UUID | FK → folders (cascade delete) |
| `storage_path` | TEXT | Path in Supabase Storage |
| `url` | TEXT | URL pubblico |
| `alt_text` | TEXT | Testo alternativo |
| `sort_order` | INT | Ordine nella cartella |
| `width` | INT | Larghezza in px |
| `height` | INT | Altezza in px |

### Sicurezza (RLS)

- **Lettura pubblica** su tutte le tabelle (folders solo con `is_visible = true`)
- **Scrittura** (INSERT, UPDATE, DELETE) solo per utente admin autenticato

---

## Struttura Progetto

```
src/
├── app/
│   ├── layout.tsx                      — Root layout (legge site_settings + content globali)
│   ├── page.tsx                        — Homepage (legge sezioni 'home' da DB)
│   ├── gallery/
│   │   ├── page.tsx                    — Griglia cartelle visibili
│   │   └── [slug]/page.tsx             — Foto della cartella + lightbox
│   └── admin/
│       ├── login/page.tsx              — Login admin
│       ├── layout.tsx                  — Layout admin protetto (sidebar)
│       ├── page.tsx                    — Dashboard
│       ├── settings/page.tsx           — Impostazioni sito (colori, font, social)
│       ├── content/page.tsx            — Gestione TUTTI i testi e immagini
│       └── folders/
│           ├── page.tsx                — Lista cartelle
│           └── [slug]/photos/page.tsx  — Gestione foto della cartella
├── components/
│   ├── ui/                             — Button, Input, Toast, Modal, ecc.
│   ├── admin/                          — Sidebar, ContentEditor, ImageUploader
│   └── public/                         — DynamicSection, MasonryGrid, Lightbox
├── lib/
│   ├── supabase/                       — Client browser, server, middleware auth
│   ├── content.ts                      — Helper: getContent(key), getContentByPage(page)
│   └── utils.ts
└── styles/
    ├── globals.css                     — Reset + design system
    ├── admin.css                       — Stili admin
    └── public.css                      — Stili sito pubblico
```

---

## Funzionalità Admin Panel

### Pagina Impostazioni (`/admin/settings`)

Modifica delle impostazioni globali del sito:
- Titolo del sito
- Colore primario e accento
- Font titoli e corpo
- Link Instagram, email, sito web

### Pagina Contenuti (`/admin/content`)

Interfaccia per modificare **ogni testo e immagine** del sito, raggruppati per pagina:

```
📄 Home
  ├─ Titolo Hero          [___testo editabile___]
  ├─ Sottotitolo Hero     [___testo editabile___]
  ├─ Immagine Hero        [🖼 preview] [Sostituisci]
  ├─ Titolo About         [___testo editabile___]
  ├─ Testo About          [___textarea___]
  ├─ Foto About           [🖼 preview] [Sostituisci]
  └─ Testo CTA            [___testo editabile___]

📄 Galleria
  ├─ Titolo Galleria      [___testo editabile___]
  └─ Sottotitolo          [___testo editabile___]

🌐 Globale
  ├─ Logo                 [🖼 preview] [Sostituisci]
  ├─ Label Nav Home       [___testo editabile___]
  ├─ Label Nav Portfolio  [___testo editabile___]
  └─ Testo Footer         [___testo editabile___]
```

### Pagina Cartelle (`/admin/folders`)

- Lista di tutte le cartelle con drag & drop per riordinare
- Creazione/modifica cartella (nome, descrizione, data evento, cover)
- Toggle visibilità pubblica/nascosta
- Eliminazione con conferma

### Pagina Foto (`/admin/folders/[slug]/photos`)

- Griglia foto della cartella
- Upload multiplo con barra di avanzamento
- Drag & drop per riordinare
- Sostituzione singola immagine (mantiene posizione e metadati)
- Eliminazione con conferma
- Compressione automatica lato client prima dell'upload

---

## Sito Pubblico

### Homepage

- **Hero section**: immagine full-width con titolo e sottotitolo sovrapposti
- **About section**: foto e bio del fotografo
- **Cartelle in evidenza**: preview delle ultime cartelle/eventi
- **CTA**: call-to-action verso la galleria completa

### Galleria (`/gallery`)

- Griglia di card con cover, titolo e data di ogni cartella visibile

### Cartella Singola (`/gallery/[slug]`)

- Griglia masonry responsive con tutte le foto
- Lightbox fullscreen con navigazione (frecce + swipe)
- Lazy loading con blur placeholder

### Layout Globale

- **Header**: logo, nome fotografo, navigazione — tutto da DB
- **Footer**: link social, testo crediti — tutto da DB
- Design responsive mobile-first
- Animazioni e micro-interazioni premium

---

## Fasi di Sviluppo

### Fase 1 — Infrastruttura & Setup
Creazione progetto Supabase, inizializzazione Next.js, configurazione client e Storage, design system base.

### Fase 2 — Database
Creazione tabelle (`site_settings`, `content_sections`, `folders`, `photos`), RLS, seed iniziale con tutti i testi e immagini di default.

### Fase 3 — Autenticazione Admin
Supabase Auth con singolo utente admin, pagina login, middleware protezione rotte, layout admin.

### Fase 4 — Admin: Impostazioni
Pagina settings per colori, font, social links.

### Fase 5 — Admin: Gestione Contenuti
Pagina per editing di tutti i testi e immagini del sito, raggruppati per pagina, con preview in tempo reale.

### Fase 6 — Admin: Gestione Cartelle
CRUD cartelle con drag & drop riordino e toggle visibilità.

### Fase 7 — Admin: Gestione Foto
Upload multiplo, riordino, sostituzione ed eliminazione foto.

### Fase 8 — Sito Pubblico: Layout & Homepage
Layout dinamico e homepage con tutti i contenuti letti da DB.

### Fase 9 — Sito Pubblico: Galleria
Pagina cartelle, pagina foto con masonry grid e lightbox.

### Fase 10 — SEO, Performance & Deploy
Meta tag dinamici, sitemap, ottimizzazione immagini, deploy su Vercel.
