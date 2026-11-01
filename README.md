# CivicTrack — Crowdsourced Civic Infrastructure Tracker

A full-stack web application that lets citizens report civic infrastructure issues (potholes, broken streetlights, garbage, water leaks, etc.) by dropping a pin on a map and uploading a photo. Administrators get a real-time dashboard to triage, assign, verify, and resolve issues. Built as a Second-Year Minor Project for the Department of Computer Engineering, ICEM Pune.

> **Project source**: Built from the project brief in `Crowdsourced_Civic_Infrastructure_Tracker.pptx` (Indira College of Engineering Management, Pune — Academic Year 2026-27).

---

## Live Preview

The app runs at the **Preview Panel** URL shown on the right of the chat interface. Click **Open in New Tab** to view it in a separate browser tab.

## Demo Accounts

| Role     | Email                 | Password     |
| -------- | --------------------- | ------------ |
| Admin    | `admin@civic.gov`     | `admin123`   |
| Citizen  | `citizen@example.com` | `citizen123` |
| Citizen  | `priya@example.com`   | `citizen123` |
| Citizen  | `rohan@example.com`   | `citizen123` |

You can also register new citizens or admins from the Sign In dialog.

---

## Tech Stack

### Frontend
- **Next.js 16** (App Router) + **React 19** + **TypeScript 5**
- **Tailwind CSS 4** with **shadcn/ui** (New York style) components
- **Leaflet** + **react-leaflet** for the interactive map (OpenStreetMap tiles — no API key required)
- **Recharts** for the admin dashboard charts
- **Zustand** for client state (auth)
- **Sonner** + Radix Toast for notifications
- **Framer Motion** for animations
- **Socket.io-client** for real-time updates

### Backend
- **Next.js API Routes** (App Router `route.ts` handlers)
- **Prisma ORM** with the **SQLite** provider (sandbox default — see "MongoDB migration" below)
- **Socket.io** mini-service (`mini-services/realtime-service/`) for real-time issue broadcasts
- Signed-cookie session auth (HMAC-SHA256 over base64url JSON payload — no external auth dep required)

### Database
- **Prisma** schema in `prisma/schema.prisma`, designed in **MongoDB document-style**:
  - `User` → users collection (citizens + admins)
  - `Issue` → issues collection with embedded location, photo, denormalized counters
  - `Vote` → votes collection (one per user per issue, compound-unique)
  - `Comment` → comments collection
  - `StatusHistory` → audit-trail collection for the timeline feature

---

## Features

### Citizen side
- Sign in / register as a citizen
- Report an issue with title, description, category, photo, and pin-on-map location
- Auto-fill address via OpenStreetMap Nominatim reverse geocoding
- Auto-detect current location via the browser Geolocation API
- Browse the community map with all reported issues
- Browse the community feed with search, status filter, category filter, and sort (recent / most-supported / priority)
- Upvote (toggle) issues to show support
- Comment on issues to start community discussion
- Bookmark issues to track them later
- View full issue detail with status timeline, comments, and location map
- Real-time updates: new issues, status changes, votes, and comments propagate instantly
- **AI Photo Categorization**: upload a photo and click "Auto-detect category with AI" — the VLM (Vision Language Model) analyzes the image and suggests a category, priority, and title
- **Nearby duplicate detection**: when you pin a location and select a category, the form warns you about similar open issues within 500m to prevent duplicates
- **Gamification**: earn points for every action (report +10, comment +5, upvote +2, issue resolved +50) and climb through 5 levels (Newcomer → Contributor → Active Citizen → Community Champion → Civic Hero)
- **Badges**: unlock 9 achievement badges (First Report, Active Reporter, Problem Solver, Community Champion, etc.)
- **User Profile**: view your stats, earned badges, level progress, and browse your reported / upvoted / bookmarked issues
- **Leaderboard**: compete with other citizens — top 3 on a podium, full rankings, and a points-earning guide
- **Dark mode**: toggle between light and dark themes

### Administrator side
- Sign in / register as an admin
- Dashboard with:
  - Stat cards: total, reported, in-progress, resolved, verified, total upvotes, total comments, avg resolution time
  - Bar chart of issues by category
  - Pie chart of issues by ward
  - Recent reports list
  - Issue management table with status update, verify, and assign actions
  - **CSV export** of all issues (with optional status/category filters)
- **Enhanced Analytics** (new):
  - 30-day trend line chart: daily reports vs resolutions
  - Category resolution rates (horizontal bar chart)
  - Average resolution time by category (hours)
  - Issue hotspots: top 8 locations with the most issues in the last 30 days
  - 7-day and 30-day summary stats
- In the issue detail page (admin only): admin actions panel for status update, assignment, and photo verification

### Real-time
- Socket.io mini-service on port 3003 (websocket) + port 3004 (internal broadcast API)
- Events: `issue:created`, `issue:updated`, `issue:vote`, `issue:comment`
- Next.js API routes POST to `127.0.0.1:3004/broadcast` after every mutation; the realtime service emits to all connected clients

---

## Project Structure

```
.
├── prisma/
│   └── schema.prisma              # Database schema (MongoDB-style)
├── scripts/
│   └── seed.ts                    # Seeds admin + citizens + 10 sample issues around Pune
├── mini-services/
│   └── realtime-service/          # Socket.io + internal broadcast API
│       ├── index.ts
│       └── package.json
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── auth/{login,logout,register,me}/route.ts
│   │   │   ├── issues/route.ts                    # GET (list+filter) / POST (create)
│   │   │   ├── issues/[id]/route.ts               # GET / PATCH (admin status update)
│   │   │   ├── issues/[id]/upvote/route.ts        # POST (toggle)
│   │   │   ├── issues/[id]/comments/route.ts      # GET / POST
│   │   │   ├── issues/[id]/timeline/route.ts      # GET status history
│   │   │   ├── issues/[id]/verify/route.ts        # POST (admin photo verification)
│   │   │   ├── issues/[id]/assign/route.ts        # POST (admin assignment)
│   │   │   └── admin/stats/route.ts               # GET dashboard stats
│   │   ├── globals.css            # Tailwind + Leaflet + civic theme
│   │   ├── layout.tsx             # Root layout with Toaster + Sonner
│   │   └── page.tsx               # Main SPA: header + 6 views + footer
│   ├── components/
│   │   ├── ui/                    # shadcn/ui component library
│   │   └── civic/
│   │       ├── auth-modal.tsx     # Sign in / register dialog
│   │       ├── issue-map.tsx      # Leaflet map with custom pin markers
│   │       ├── issue-card.tsx     # Compact issue card for feeds
│   │       ├── report-form.tsx    # Report-an-issue form with map + photo
│   │       ├── issue-detail.tsx   # Full issue view + timeline + comments + admin actions
│   │       ├── feed-view.tsx      # Filterable / sortable community feed
│   │       ├── home-view.tsx      # Landing page: hero, how it works, features, recent
│   │       ├── admin-dashboard.tsx# Admin stats + charts + management table + analytics
│   │       ├── profile-view.tsx    # User profile with stats, badges, level, activity tabs
│   │       ├── leaderboard-view.tsx# Community leaderboard with podium + rankings
│   │       └── theme-toggle.tsx    # Dark/light mode toggle
│   ├── hooks/
│   │   ├── use-auth.ts            # Zustand auth store
│   │   └── use-realtime.ts        # Socket.io subscription hook
│   └── lib/
│       ├── auth.ts                # Password hashing + signed-cookie sessions
│       ├── categories.ts          # Categories + statuses + priorities (single source of truth)
│       ├── client-utils.ts        # timeAgo, formatDateTime, fileToDataUrl, reverseGeocode
│       ├── db.ts                  # Prisma client singleton
│       ├── gamification.ts        # Points, levels, badges definitions
│       ├── gamification-service.ts# Points awarding + badge checking logic
│       └── types.ts               # Shared TS types
├── Caddyfile                      # Gateway: routes /?XTransformPort=<port> to that port
└── package.json
```

---

## API Reference

| Method | Endpoint                              | Auth        | Description                              |
| ------ | ------------------------------------- | ----------- | ---------------------------------------- |
| POST   | `/api/auth/register`                  | public      | Register a new citizen or admin          |
| POST   | `/api/auth/login`                     | public      | Sign in with email + password            |
| POST   | `/api/auth/logout`                    | public      | Clear session cookie                     |
| GET    | `/api/auth/me`                        | public      | Get current session user (with points + level) |
| GET    | `/api/issues`                         | public      | List issues with filters/sort/pagination |
| POST   | `/api/issues`                         | citizen+    | Create a new issue                       |
| GET    | `/api/issues/[id]`                    | public      | Get issue detail with comments + history |
| PATCH  | `/api/issues/[id]`                    | admin       | Update status / priority                 |
| POST   | `/api/issues/[id]/upvote`             | citizen+    | Toggle upvote (awards points)            |
| GET    | `/api/issues/[id]/comments`           | public      | List comments                            |
| POST   | `/api/issues/[id]/comments`           | citizen+    | Add a comment (awards points)            |
| GET    | `/api/issues/[id]/timeline`           | public      | Get status history                       |
| POST   | `/api/issues/[id]/verify`             | admin       | Verify + mark resolved                   |
| POST   | `/api/issues/[id]/assign`             | admin       | Assign to a worker / department          |
| POST   | `/api/issues/[id]/bookmark`           | citizen+    | Toggle bookmark (awards points)          |
| GET    | `/api/issues/[id]/nearby`             | public      | Find similar issues within radius        |
| GET    | `/api/admin/stats`                    | admin       | Dashboard statistics                     |
| GET    | `/api/admin/analytics`                | admin       | Enhanced analytics (trends, hotspots)    |
| GET    | `/api/admin/export`                   | admin       | Export issues as CSV                     |
| GET    | `/api/user/profile`                   | citizen+    | Current user's profile + stats + badges  |
| GET    | `/api/user/issues?type=reported\|upvoted\|bookmarked` | citizen+ | User's issues by type |
| GET    | `/api/leaderboard`                    | public      | Top citizens by points                   |
| POST   | `/api/ai/categorize`                  | citizen+    | AI photo categorization (VLM)            |

### Query parameters for `GET /api/issues`
- `status` — comma-separated statuses (`REPORTED,IN_PROGRESS`)
- `category` — comma-separated category IDs (`POTHOLE,GARBAGE`)
- `ward` — ward name
- `sort` — `recent` (default) | `upvotes` | `priority`
- `search` — text search across title/description/address
- `limit`, `offset` — pagination (max 500)

---

## Local Development

```bash
# Install dependencies
bun install

# Push the Prisma schema to the database
bun run db:push

# Seed sample data (1 admin, 3 citizens, 10 issues around Pune)
bun run scripts/seed.ts

# Start the realtime service (separate terminal)
cd mini-services/realtime-service
bun install
bun run dev

# The Next.js dev server starts automatically on port 3000
# (the sandbox auto-runs `bun run dev`)

# Lint
bun run lint
```

---

## MongoDB Migration

The schema is intentionally written in MongoDB document-style so it can be migrated to MongoDB with minimal changes:

1. Provision a MongoDB instance (Atlas, local, or Docker).
2. In `prisma/schema.prisma`, change the datasource provider:
   ```prisma
   datasource db {
     provider = "mongodb"
     url      = env("DATABASE_URL")   # e.g. mongodb+srv://user:pass@cluster/db
   }
   ```
3. (Optional) For native ObjectId fields, replace `@default(cuid())` with `@db.ObjectId` on the `id` field of each model and on foreign-key fields. The schema already uses `String` for all IDs, which also works on MongoDB.
4. Run `bun run db:push` to push the schema to MongoDB.
5. Re-run `bun run scripts/seed.ts` to populate sample data.

The Prisma Client API is identical across SQLite and MongoDB — no application code changes are required.

---

## Categories

| ID           | Label              | Icon | Default Priority |
| ------------ | ------------------ | ---- | ---------------- |
| POTHOLE      | Pothole            | 🕳️   | HIGH             |
| STREETLIGHT  | Street Light       | 💡   | MEDIUM           |
| GARBAGE      | Garbage / Waste    | 🗑️   | MEDIUM           |
| WATER        | Water Leak         | 💧   | HIGH             |
| SEWAGE       | Sewage / Drainage  | 🚱   | HIGH             |
| ROAD         | Road Damage        | 🛣️   | HIGH             |
| TREE         | Fallen Tree        | 🌳   | MEDIUM           |
| SIGNAGE      | Broken Signage     | 🚦   | LOW              |
| GRAFFITI     | Graffiti           | 🎨   | LOW              |
| OTHER        | Other              | 📍   | MEDIUM           |

## Statuses

`REPORTED` → `IN_PROGRESS` → `RESOLVED` (or `REJECTED`)

Every status change is recorded in the `StatusHistory` collection and rendered as a timeline on the issue detail page.

---

## Unique Selling Points (per project brief)

1. **Geolocation tagging** — accurate, map-based plotting of every reported issue
2. **Community upvoting** — admins can prioritise high-impact issues
3. **Transparent status timeline** — Reported → In Progress → Resolved, visible to every citizen
4. **Automated photo-verification** — admins confirm resolution with on-site verification
5. **Single unified dashboard** — connects citizens and local administrators in real time

---

## Future Scope (per project brief) — Implementation Status

- ~~Native mobile app for Android and iOS with offline reporting support~~ — *not yet implemented (web-only for now)*
- ✅ **AI-based auto-categorization and severity detection from uploaded photos** — *implemented via VLM (Vision Language Model) — click "Auto-detect category with AI" in the report form*
- SMS and email notifications for status changes on reported issues — *not yet implemented*
- Integration with municipal work-order systems for automated task assignment — *partial: admins can assign issues to workers/departments*
- ✅ **Analytics dashboard for administrators to identify recurring problem areas over time** — *implemented: 30-day trend chart, category resolution rates, avg resolution time, issue hotspots*
- ✅ **Gamification and rewards to encourage active citizen participation** — *implemented: points system, 5 levels, 9 badges, leaderboard, user profile with stats*
