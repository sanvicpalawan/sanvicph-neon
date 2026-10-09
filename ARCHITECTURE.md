# SANVIC 2027 — Architecture & Code Tree

## High-Level Structure

```
sanvicph-neon/
├── app/                          # Next.js App Router
│   ├── admin/                    # Admin dashboard pages
│   │   └── page.tsx
│   ├── api/                      # API Routes (Server)
│   │   ├── admin/                # Admin-only endpoints
│   │   │   ├── data/route.ts             # Full admin data (GET/PUT/DELETE)
│   │   │   ├── locations/import-kmz/route.ts
│   │   │   ├── media/route.ts            # Media upload/delete
│   │   │   ├── owners/route.ts           # Owner requests review
│   │   │   └── session/route.ts          # Admin PIN auth
│   │   ├── media/[id]/route.ts           # Public media streaming
│   │   ├── owners/                       # Owner portal endpoints
│   │   │   ├── route.ts                  # Owner dashboard data
│   │   │   ├── uploads/route.ts          # Owner upload tickets
│   │   │   └── uploads/[id]/route.ts     # Owner upload PUT
│   │   ├── public/                       # Public read endpoints
│   │   │   ├── journey/route.ts          # Traveler journey data
│   │   │   └── site/route.ts             # Full site content (map, places, media)
│   │   └── travelers/                    # Traveler endpoints
│   │       ├── route.ts                  # Traveler auth/profile
│   │       ├── joins/route.ts            # Join opportunities
│   │       ├── uploads/route.ts          # Traveler upload tickets
│   │       ├── uploads/[id]/route.ts     # Traveler upload PUT/GET
│   │       └── visits/route.ts           # Community visits
│   ├── owners/page.tsx                   # Owner portal page (client)
│   ├── page.tsx                          # Home page (client)
│   ├── layout.tsx                        # Root layout
│   ├── globals.css                       # Global styles
│   └── owner-public.css                  # Owner portal styles
├── components/                           # React Components
│   ├── admin/                            # Admin UI
│   │   ├── admin-app.tsx                 # Admin shell
│   │   ├── inline-media-picker.tsx
│   │   ├── kmz-import-panel.tsx
│   │   ├── links-editor.tsx
│   │   ├── location-map-editor.tsx
│   │   ├── owners-panel.tsx
│   │   ├── rooms-editor.tsx
│   │   └── traveler-media-panel.tsx
│   ├── owners/                           # Owner portal UI
│   │   ├── owner-portal.tsx              # Main owner portal (client)
│   │   └── owners.css
│   ├── ui/                               # Radix-based UI primitives
│   │   ├── accordion.tsx ... marker.tsx  # 40+ components
│   ├── lightbox.tsx                      # Image/video lightbox
│   ├── location-mini-map.tsx             # Small map preview
│   ├── onboarding.tsx                    # First-run flow
│   ├── public-media.tsx                  # Public media display
│   ├── sanvic-app.tsx                    # Main app shell (client)
│   ├── sanvic-map.tsx                    # Leaflet map component
│   └── traveler-tools.tsx                # Traveler upload/join UI
├── lib/                                  # Server/Shared Logic
│   ├── db/                               # Database Layer
│   │   ├── neon.ts                       # Drizzle + Neon client
│   │   └── schema.ts                     # 13-table schema (Drizzle)
│   ├── storage/                          # Storage Layer
│   │   └── local.ts                      # Local filesystem ops
│   ├── admin-server.ts                   # Admin auth, audit, seeding
│   ├── traveler-server.ts                # Traveler auth/sessions
│   ├── owner-server.ts                   # Owner auth/permissions
│   ├── owner-types.ts                    # Owner Zod schemas
│   ├── cms-types.ts                      # Public content types
│   ├── sanvic-data.ts                    # Static seed data
│   ├── kmz-parser.ts                     # KMZ import parser
│   ├── kmz-seed.ts                       # KMZ seed data
│   ├── admin-seed.ts                     # Admin seed data
│   ├── place-links.ts                    # Place link icons
│   └── utils.ts                          # Helpers
├── public/                               # Static Assets
│   ├── photo-credits.json
│   ├── sanvic-logo.png
│   └── gcash-sanvic-pro.jpeg
├── drizzle.config.ts                     # Drizzle Kit config
├── next.config.ts                        # Next.js config
├── package.json
├── tsconfig.json
├── README.md
├── TECHSTACK.md
└── ARCHITECTURE.md
```

## Data Flow

### Public Site Load (`/api/public/site`)
```
GET /api/public/site
  → seedDefaults() if empty
  → Parallel fetch: site_content, content_items, places, media, traveler_experiences
  → Transform → JSON response (copy, communities, categories, opportunities, places, media, travelerExperiences)
  → sanvic-app.tsx renders map + sheets
```

### Admin Data (`/api/admin/data`)
```
GET /api/admin/data
  → requireAdmin() via cookie
  → seedDefaults()
  → Full data dump for admin UI (includes drafts, audit log, all media)
```

### Media Upload (Admin)
```
POST /api/admin/media (multipart)
  → requireAdmin()
  → Save file to /app/data/uploads/media/{uuid}.{ext}
  → Insert media row (objectKey = path)
  → Return media object with /api/media/{id} URL
```

### Media Streaming
```
GET /api/media/{id}
  → Fetch media/owner_uploads row by id
  → Read file from /app/data/uploads/{objectKey}
  → Stream with proper Content-Type, caching headers
```

### Owner Portal
```
GET /api/owners
  → ownerUser() via JWT or open-builder mode
  → Fetch: managed places, requests, uploads, directory
  → Sign URLs for owner uploads → /api/owners/uploads/{id}

POST /api/owners/uploads
  → Create upload ticket (objectKey = owner-private/{userId}/{uuid}.{ext})
  → Return PUT URL → /api/owners/uploads/{id}

PUT /api/owners/uploads/{id}
  → Receive file, write to local disk
  → Mark owner_uploads.completed = true
```

### Traveler Uploads
```
POST /api/travelers/uploads
  → requireTraveler() via cookie
  → Create traveler_experiences row (status=pending)
  → Return PUT URL → /api/travelers/uploads/{id}

PUT /api/travelers/uploads/{id}
  → Receive file, write to local disk
  → Mark completed, status=pending
```

## Database Schema (13 Tables)

| Table | Purpose |
|-------|---------|
| `admin_sessions` | Admin PIN sessions |
| `site_content` | Key-value content (copy, settings) |
| `content_items` | Communities, categories, opportunities, badges, onboarding |
| `places` | Locations (published/draft, rooms, links, photos) |
| `audit_log` | Admin action trail |
| `location_managers` | Owner ↔ place mapping |
| `travelers` | Device-bound traveler identities |
| `traveler_sessions` | Traveler cookie sessions |
| `media` | Admin-uploaded media |
| `traveler_experiences` | Traveler photo uploads |
| `traveler_joins` | Traveler → opportunity joins |
| `traveler_visits` | Traveler → place visits |
| `community_visits` | Traveler → community visits |
| `owner_requests` | Claim/changes/payment requests |
| `owner_uploads` | Owner portal uploads |

## Key Design Decisions

1. **No Supabase** — Neon (Postgres) + local filesystem only
2. **Cookie-based auth** — No JWT in localStorage, HttpOnly cookies
3. **API-first** — All data via API routes, client components fetch
4. **Standalone output** — `next.config.ts` output: 'standalone' for Docker
5. **Type-safe DB** — Drizzle schema → TypeScript types auto-inferred
6. **Seed on demand** — `seedDefaults()` creates tables/data if missing