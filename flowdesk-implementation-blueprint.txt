# Flowdesk — Implementation Blueprint

**Purpose:** This document is the bridge between architecture (Phases 0–9) and code. It has zero code — only structure, responsibilities, and flow — so that when Increment 1 begins, every file has a known purpose before it's created.

---

## 1. Complete Folder Structure

```
/flowdesk
├── apps/
│   ├── server/
│   │   ├── src/
│   │   │   ├── config/
│   │   │   │   ├── db.js
│   │   │   │   ├── env.js
│   │   │   │   └── constants.js
│   │   │   ├── models/
│   │   │   │   ├── User.js
│   │   │   │   ├── Workspace.js
│   │   │   │   ├── Board.js
│   │   │   │   ├── Column.js
│   │   │   │   ├── Card.js
│   │   │   │   ├── Note.js
│   │   │   │   └── Label.js
│   │   │   ├── routes/
│   │   │   │   ├── auth.routes.js
│   │   │   │   ├── workspace.routes.js
│   │   │   │   ├── board.routes.js
│   │   │   │   ├── column.routes.js
│   │   │   │   ├── card.routes.js
│   │   │   │   ├── note.routes.js
│   │   │   │   ├── label.routes.js
│   │   │   │   ├── search.routes.js
│   │   │   │   └── index.js
│   │   │   ├── controllers/
│   │   │   │   ├── auth.controller.js
│   │   │   │   ├── workspace.controller.js
│   │   │   │   ├── board.controller.js
│   │   │   │   ├── column.controller.js
│   │   │   │   ├── card.controller.js
│   │   │   │   ├── note.controller.js
│   │   │   │   ├── label.controller.js
│   │   │   │   └── search.controller.js
│   │   │   ├── services/
│   │   │   │   ├── auth.service.js
│   │   │   │   ├── workspace.service.js
│   │   │   │   ├── board.service.js
│   │   │   │   ├── column.service.js
│   │   │   │   ├── card.service.js
│   │   │   │   ├── note.service.js
│   │   │   │   ├── label.service.js
│   │   │   │   ├── search.service.js
│   │   │   │   └── position.service.js
│   │   │   ├── middleware/
│   │   │   │   ├── auth.middleware.js
│   │   │   │   ├── errorHandler.js
│   │   │   │   ├── requestLogger.js
│   │   │   │   ├── softDelete.middleware.js
│   │   │   │   └── rateLimiter.js
│   │   │   ├── utils/
│   │   │   │   ├── AppError.js
│   │   │   │   ├── logger.js
│   │   │   │   └── tiptapToText.js
│   │   │   ├── validators/
│   │   │   │   ├── auth.validator.js
│   │   │   │   ├── workspace.validator.js
│   │   │   │   ├── board.validator.js
│   │   │   │   ├── column.validator.js
│   │   │   │   ├── card.validator.js
│   │   │   │   ├── note.validator.js
│   │   │   │   └── label.validator.js
│   │   │   ├── app.js
│   │   │   └── server.js
│   │   ├── tests/
│   │   │   ├── unit/
│   │   │   └── integration/
│   │   ├── .env.example
│   │   └── package.json
│   │
│   └── client/
│       ├── app/
│       │   ├── login/
│       │   │   └── page.jsx
│       │   ├── (app)/
│       │   │   ├── layout.jsx
│       │   │   ├── workspace/[workspaceId]/
│       │   │   │   ├── page.jsx
│       │   │   │   ├── board/[boardId]/page.jsx
│       │   │   │   └── notes/[noteId]/page.jsx
│       │   │   └── search/page.jsx
│       │   ├── layout.jsx
│       │   └── globals.css
│       ├── components/
│       │   ├── board/
│       │   │   ├── BoardView.jsx
│       │   │   ├── Column.jsx
│       │   │   ├── Card.jsx
│       │   │   └── CardDetailModal.jsx
│       │   ├── notes/
│       │   │   ├── NoteEditor.jsx
│       │   │   └── NoteList.jsx
│       │   ├── workspace/
│       │   │   ├── WorkspaceSwitcher.jsx
│       │   │   └── Sidebar.jsx
│       │   ├── search/
│       │   │   └── SearchModal.jsx
│       │   └── ui/
│       │       ├── Button.jsx
│       │       ├── Modal.jsx
│       │       ├── Input.jsx
│       │       ├── Toast.jsx
│       │       └── QueryBoundary.jsx
│       ├── hooks/
│       │   ├── queries/
│       │   │   ├── useWorkspaces.js
│       │   │   ├── useBoards.js
│       │   │   ├── useColumns.js
│       │   │   ├── useCards.js
│       │   │   ├── useNotes.js
│       │   │   ├── useLabels.js
│       │   │   └── useSearch.js
│       │   └── mutations/
│       │       ├── useCreateCard.js
│       │       ├── useMoveCard.js
│       │       ├── useUpdateCard.js
│       │       ├── useDeleteCard.js
│       │       └── (equivalent files per resource)
│       ├── stores/
│       │   ├── useUIStore.js
│       │   └── useActiveWorkspaceStore.js
│       ├── lib/
│       │   ├── apiClient.js
│       │   └── queryKeys.js
│       ├── .env.example
│       └── package.json
│
├── .github/workflows/ci.yml
├── package.json
├── .eslintrc.json
├── .prettierrc
├── .gitignore
└── README.md
```

---

## 2. Folder Responsibilities

| Folder | Responsibility |
|---|---|
| `server/src/config` | Environment loading, DB connection, app-wide constants. No business logic. |
| `server/src/models` | Mongoose schemas only — shape and validation of stored data. No queries beyond schema-level hooks. |
| `server/src/routes` | URL → controller wiring. No logic, no validation, no DB access. |
| `server/src/controllers` | Parse request, call validator, call service, shape response. No business rules. |
| `server/src/services` | All business logic: position math, cascade deletes, search assembly, auth logic. The only layer allowed to compose multiple model operations. |
| `server/src/middleware` | Cross-cutting request concerns: auth check, error formatting, logging, soft-delete filtering, rate limiting. |
| `server/src/utils` | Stateless helpers used across layers (error class, logger instance, TipTap text extraction). |
| `server/src/validators` | Zod schemas per resource — the single source of truth for what a valid request body looks like. |
| `client/app` | Route definitions only (App Router). Pages compose components; they don't contain business logic. |
| `client/components` | Presentational + container components, grouped by domain. |
| `client/hooks/queries` | One TanStack Query hook per resource — owns query key + fetch function. |
| `client/hooks/mutations` | One hook per write operation — owns the mutation, its optimistic update, and cache invalidation. |
| `client/stores` | Zustand stores — UI-only state, never server data. |
| `client/lib` | API client (fetch wrapper), shared query key factory. |

---

## 3. Responsibilities of Every File (Server, by increment)

### Increment 1 — Foundation
- `config/db.js` — connects Mongoose to Atlas, exports connection promise
- `config/env.js` — validates required env vars exist at boot (fail fast, not at first request)
- `app.js` — Express app instance, middleware registration order, route mounting
- `server.js` — imports `app.js`, starts HTTP listener
- `middleware/errorHandler.js` — last-registered middleware; catches `AppError` + unexpected errors
- `middleware/requestLogger.js` — logs method/path/status/duration per request via Pino
- `utils/AppError.js` — `class AppError extends Error { constructor(statusCode, message) }`
- `utils/logger.js` — configured Pino instance, exported singleton

### Increment 2 — Auth
- `models/User.js` — email, passwordHash, name
- `services/auth.service.js` — hash/compare password, issue/verify JWT
- `controllers/auth.controller.js` — login/logout request handling, sets/clears cookie
- `middleware/auth.middleware.js` — reads JWT from cookie, verifies, attaches `req.userId`, else 401
- `validators/auth.validator.js` — Zod schema for login payload
- `middleware/rateLimiter.js` — applied to `/api/auth/login`

### Increment 3 — Workspaces (establishes the CRUD pattern reused by 4–7)
- `models/Workspace.js`
- `services/workspace.service.js` — create/list/rename/soft-delete + cascade trigger
- `controllers/workspace.controller.js`
- `routes/workspace.routes.js`
- `validators/workspace.validator.js`
- `middleware/softDelete.middleware.js` — Mongoose `pre('find')` hook pattern, applied per-schema

### Increment 4 — Boards & Columns
- `models/Board.js`, `models/Column.js`
- `services/board.service.js`, `services/column.service.js`
- `services/position.service.js` — shared float-position insert/reorder math, used by Column and Card
- Corresponding controllers/routes/validators

### Increment 5 — Cards
- `models/Card.js`
- `services/card.service.js` — includes move logic (columnId + position change)
- Corresponding controller/routes/validator

### Increment 6 — Labels
- `models/Label.js`, `services/label.service.js`, controller/routes/validator

### Increment 7 — Notes
- `models/Note.js`
- `utils/tiptapToText.js` — extracts plain text from TipTap JSON for `contentText`
- `services/note.service.js` — handles `linkedCardIds` array operations
- Corresponding controller/routes/validator

### Increment 8 — Search
- `services/search.service.js` — parallel `$text` query across Card + Note, merges/ranks/groups
- `controllers/search.controller.js`, `routes/search.routes.js`

---

## 4. Data Flow Diagram — Card Move (Drag-and-Drop)

```
User drags card in browser
        │
        ▼
dnd-kit onDragEnd fires (client-only, no network yet)
        │
        ▼
Client computes new float position
   (between target siblings)
        │
        ▼
useMoveCard() mutation fires:
  1. Optimistically updates TanStack Query cache (instant UI)
  2. Sends PATCH /api/cards/:id { columnId, position }
        │
        ▼
Express: router → auth.middleware (verify JWT)
        │
        ▼
card.controller → card.validator (Zod: validate body)
        │
        ▼
card.service.moveCard()
   - validates target column belongs to same board
   - writes new columnId + position to Card doc
        │
        ▼
Response returns updated Card
        │
        ▼
On success: cache already correct (optimistic update confirmed)
On failure: cache rolled back, toast shown (Phase 5 pattern)
```

## 5. API Flow — General Request Lifecycle

```
Request
  → requestLogger (log start)
  → CORS check
  → auth.middleware (except /auth/login, /health)
  → route → controller
  → validator (Zod parse; throws AppError(400) on failure)
  → service (business logic, DB ops via Mongoose)
  → controller formats response
  → (on any thrown error, anywhere above) → errorHandler
  → requestLogger (log end, status, duration)
  → Response sent
```

## 6. Component Hierarchy (Board View)

```
app/(app)/workspace/[workspaceId]/board/[boardId]/page.jsx
 └── BoardView
      ├── Sidebar (workspace switcher, nav)
      ├── Column (×N, sortable via dnd-kit)
      │    └── Card (×N, draggable)
      │         └── (on click) CardDetailModal
      │              ├── labels editor (uses useLabels query)
      │              └── linked notes (uses useNotes query, filtered by linkedCardIds)
      └── QueryBoundary (wraps the whole view — loading/error states)
```

## 7. Database Interaction Flow (Workspace Cascade Soft-Delete)

```
DELETE /api/workspaces/:id
  → workspace.controller
  → workspace.service.softDeleteWorkspace(id)
       1. Set Workspace.deletedAt = now
       2. Set deletedAt = now on all Boards where workspaceId = id
       3. Set deletedAt = now on all Columns where workspaceId = id
       4. Set deletedAt = now on all Cards where workspaceId = id
       5. Set deletedAt = now on all Notes where workspaceId = id
       (all as a single Mongoose transaction/session where possible,
        so a partial cascade failure doesn't leave orphaned "live" children)
```

---

## 8. Development Order & Day-by-Day Plan

This assumes solo, part-time development (evenings/weekends alongside your day job) — so "days" here mean **focused working sessions**, not calendar days. Adjust pace freely; the *order* is what matters architecturally.

| Day | Increment | Focus |
|---|---|---|
| 1 | 1 | Monorepo scaffold, npm workspaces, ESLint/Prettier, Express + Next.js boot, Atlas connection |
| 2 | 1 | Health-check route, error handler + logger middleware wired, confirm both apps run via one root script |
| 3 | 2 | User model, bcrypt, JWT issuance, login/logout routes |
| 4 | 2 | Auth middleware, frontend login page, `(app)` route guard — milestone: protected page reachable |
| 5 | 3 | Workspace model + service + soft-delete middleware pattern |
| 6 | 3 | Workspace controller/routes/validator, frontend switcher + CRUD UI — milestone: workspace CRUD complete |
| 7 | 4 | Board + Column models, `position.service.js` float-math logic (unit-testable in isolation even before hardening pass) |
| 8 | 4 | Board/Column controllers/routes, frontend board view + column reorder |
| 9 | 5 | Card model + service incl. move logic |
| 10 | 5 | Card controller/routes, dnd-kit integration, optimistic move mutation — milestone: full Kanban works |
| 11 | 5 | Card detail modal, remaining card fields (due date, description) |
| 12 | 6 | Label model/service/routes + frontend label management/assignment |
| 13 | 7 | Note model, `tiptapToText.js`, note service incl. `linkedCardIds` |
| 14 | 7 | Note controller/routes, TipTap editor integration, link-to-card UI — milestone: notes complete |
| 15 | 8 | Text indexes, search service (parallel query + merge), search endpoint |
| 16 | 8 | Search modal frontend — milestone: global search works |
| 17–18 | 9 | Hardening: unit tests (position math, cascade delete, tiptapToText), integration tests (auth, CRUD routes) |
| 19 | 9 | Hardening: rate limiting, CORS lockdown, error boundary, loading-state polish |
| 20 | Deploy | Vercel + Render + Atlas prod cluster, GitHub Actions CI, README, live URL |

**Milestone checkpoints** (natural pause/demo points): end of Day 4 (auth works), Day 6 (workspace CRUD), Day 11 (Kanban complete), Day 14 (notes complete), Day 16 (search complete), Day 20 (deployed).

---

*This blueprint is the final reference before code generation begins. Increment 1 code will follow this document's file list exactly, in the order specified.*
