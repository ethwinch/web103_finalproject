# Technical Specification (Milestone 1)

## 1) System Overview

Collector's Corner is a full-stack media library platform where users can:
- browse shared catalog entries and public collections,
- add catalog media to their personal library,
- organize entries with custom tags,
- and (based on role) create new media entries with cover uploads.

The architecture follows a React SPA + Express REST API + PostgreSQL design.

## 2) Role Model and Access Matrix

| Capability | Visitor | Collector | Creator | Curator |
|---|---:|---:|---:|---:|
| Browse public collections | ✅ | ✅ | ✅ | ✅ |
| View global media catalog | ✅ | ✅ | ✅ | ✅ |
| Manage personal library entries | ❌ | ✅ | ✅ | ✅ |
| Create/manage personal tags | ❌ | ✅ | ✅ | ✅ |
| Create new global media records | ❌ | ❌ | ✅ | ✅ |
| Upload custom media covers | ❌ | ❌ | ✅ | ✅ |
| Curation-focused metadata improvements | ❌ | ❌ | ❌ | ✅ |

Authorization must combine:
1. **Role checks** (is this role allowed to perform this action?),
2. **Ownership checks** (does this user own this resource?).

## 3) API Design (RESTful)

Base path: `/api`

### Auth
- `POST /auth/register` — create user + profile
- `POST /auth/login` — establish session
- `POST /auth/logout` — destroy session
- `GET /auth/session` — return current authenticated user/session snapshot

### Users and Profiles
- `GET /users/:id` — public profile + collection summary (public-safe fields)
- `GET /me/profile` — current user's profile
- `PATCH /me/profile` — update current user's profile

### Media Catalog
- `GET /media` — list/search/filter media (public)
- `GET /media/:id` — media details (public)
- `POST /media` — create media (Creator/Curator)
- `PATCH /media/:id` — update media (Creator/Curator with policy)
- `DELETE /media/:id` — restricted delete policy (likely Curator/admin-only if implemented)

### Personal Library
- `GET /me/library` — current user's library
- `POST /me/library` — add media to personal library
- `PATCH /me/library/:entryId` — update status/rating/notes
- `DELETE /me/library/:entryId` — remove entry

### Tags
- `GET /me/tags` — list current user's tags
- `POST /me/tags` — create tag
- `PATCH /me/tags/:tagId` — rename/recolor tag
- `DELETE /me/tags/:tagId` — delete tag

### Library Entry Tag Assignment
- `POST /me/library/:entryId/tags/:tagId` — assign tag to entry
- `DELETE /me/library/:entryId/tags/:tagId` — remove tag from entry

## 4) Validation Strategy

All POST/PATCH requests validate **before** DB writes.

### Request-Level Validation
- Required fields present (for example `title`, `media_type`, `email`, `password`).
- Type checks (string, number, URL format where relevant).
- Enum checks:
  - `role` in `collector|creator|curator` (visitor is unauthenticated state, not stored role for signed-in actions unless chosen by design),
  - `media_type` in supported set,
  - `status` in allowed library states.
- Numeric bounds:
  - rating integer range (for example 1–5).

### Relational Validation
- Foreign keys must exist (`media_id`, `tag_id`, `library_entry_id`).
- Ownership checks:
  - user can only mutate their own `library_entries` and `tags`.
- Duplicate guards:
  - prevent duplicate `(user_id, media_id)` library entries,
  - prevent duplicate `(user_id, tag_name)` tags,
  - prevent duplicate `(library_entry_id, tag_id)` assignments.

### Error Response Contract
Consistent shape:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": [
      { "field": "title", "issue": "Title is required" }
    ]
  }
}
```

## 5) Authentication and Session Flow

Session-based auth stack:
- `express-session` middleware
- `connect-pg-simple` session store in PostgreSQL
- HTTP-only session cookie

### Login Flow
1. User submits credentials.
2. Server verifies password hash.
3. Server stores `userId` and role in session.
4. Client receives authenticated session state from `/auth/session`.

### Logout Flow
1. Client requests `/auth/logout`.
2. Server destroys session + clears cookie.
3. Client transitions to Guest/Visitor state and redirects to public page.

### Protected Route Strategy
- Backend middleware: reject unauthenticated mutating requests with 401.
- Frontend route guards: redirect to login for protected pages.
- Role middleware: reject unauthorized actions with 403.

## 6) Frontend Architecture

## Route Groups
- Public:
  - `/` (landing/discovery)
  - `/media`
  - `/media/:mediaId`
  - `/users/:userId/collection`
- Protected:
  - `/my-library`
  - `/my-tags`
  - `/create-media`
  - `/profile`

### Component Hierarchy (High-Level)
- `AppLayout`
  - `TopNav` (auth controls, global navigation)
  - Route pages:
    - `MediaCatalogPage`
      - `MediaFilters`
      - `MediaGrid`
      - `MediaCard`
    - `MyLibraryPage`
      - `LibraryToolbar` (sort/filter)
      - `LibraryGrid`
      - `LibraryEntryCard`
      - `EntryEditModal` / `SlideOutPane`
    - `TagManagerPage`
      - `TagList`
      - `TagFormModal`
    - `CreateMediaPage` (Creator/Curator)
      - `MediaForm`
      - `CoverUploadField`

### Same-Page Interactions
- Tag filter chips update visible library entries without full-page reload.
- Modal-based create/edit workflows update local list state after successful mutation.

### UX Reliability Features
- Loading indicators for all asynchronous list/detail pages.
- Toast notifications for success/failure of writes.
- Empty-state messaging for no results and no personal data.

## 7) File/Image Upload Strategy (Cloudinary)

Recommended flow:
1. Client selects image and submits multipart form.
2. Backend validates mime type + size.
3. Backend uploads to Cloudinary.
4. Backend saves secure Cloudinary URL in `media.cover_image_url`.

Security considerations:
- Restrict accepted file types (jpeg/png/webp).
- Enforce max size limits.
- Never expose API secrets in frontend code.

## 8) Database Constraints and Integrity Notes

- Foreign keys with `ON DELETE CASCADE` where orphan cleanup is desired (for join rows, profile rows).
- Composite PK on `library_entry_tags` for deduplication.
- Unique constraints supporting core business rules.
- Timestamp columns for auditing and future moderation tooling.

## 9) Deployment Notes (Render)

- Separate services:
  - frontend static site,
  - backend web service,
  - managed PostgreSQL instance.
- Environment variables:
  - `DATABASE_URL`
  - `SESSION_SECRET`
  - `CLOUDINARY_*`
  - `CLIENT_ORIGIN`
- CORS and cookie settings must be environment-aware (local vs production).

## 10) Milestone 1 to Milestone 2 Handoff

Implementation should prioritize:
1. DB schema + migrations for all six core tables.
2. Auth/session infrastructure and protected route middleware.
3. Core CRUD for media, library entries, and tags.
4. Frontend flows for browse + personal library management.
5. Role-based feature gating and upload integration.
