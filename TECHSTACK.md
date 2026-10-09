# SANVIC 2027 — Tech Stack

## Core
- **Framework:** Next.js 16 (App Router, Turbopack)
- **Language:** TypeScript (strict), React 19
- **Styling:** Tailwind CSS 4, CSS Modules
- **Build:** Turbopack, ESLint 9

## Database & ORM
- **Database:** Neon (managed PostgreSQL, serverless)
- **ORM:** Drizzle ORM (type-safe, zero-runtime)
- **Migrations:** Drizzle Kit
- **Connection:** `@neondatabase/serverless` (HTTP/WebSocket)

## Storage
- **Files:** Local filesystem (`/app/data/uploads/`)
- **Served via:** Next.js API routes (`/api/media/[id]`, `/api/owners/uploads/[id]`, `/api/travelers/uploads/[id]`)
- **No external S3/R2/MinIO required**

## Map & Geospatial
- **Map Library:** Leaflet 1.9 + Leaflet.markercluster
- **Projection:** EPSG:3857 (Web Mercator)
- **Base Tiles:** Esri World Imagery (satellite)
- **Boundaries:** PSGC 2023 (converted to GeoJSON, bundled)

## Auth & Sessions
- **Admin:** Cookie-based (HttpOnly, Secure, SameSite=Strict), PIN-gated
- **Travelers:** Cookie-based (1-year TTL, device-bound)
- **Owners:** Email OTP (magic link) + JWT verification

## Deployment
- **Platform:** Coolify on UpCloud VPS (vps01.merqato.digital, SG-SIN1)
- **Container:** Docker (Next.js standalone output)
- **Reverse Proxy:** Coolify (Traefik) → Cloudflare DNS
- **Env:** `DATABASE_URL`, `ADMIN_PIN`, `UPLOAD_DIR`

## Key Dependencies
| Package | Purpose |
|---------|---------|
| `next` | App framework |
| `react` / `react-dom` | UI |
| `drizzle-orm` / `@neondatabase/serverless` | Database |
| `drizzle-kit` | Migrations |
| `leaflet` / `leaflet.markercluster` | Map |
| `zod` | Validation |
| `lucide-react` | Icons |
| `date-fns` | Dates |
| `fast-xml-parser` | KMZ/KML import |
| `recharts` | Admin charts |
| `sonner` | Toasts |

## Removed (from Supabase version)
- `@supabase/supabase-js`
- Supabase Storage (replaced by local filesystem)
- Supabase Auth (replaced by cookie/JWT)
- Supabase Realtime (not used)