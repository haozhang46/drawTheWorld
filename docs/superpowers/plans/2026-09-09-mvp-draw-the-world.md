# Draw The World MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship a mobile MVP where an Author uploads an Asset, optionally creates a Draft on the map, establishes a Marker Anchor on site, and Saves a World Placement that Viewers discover via Nearby and see in AR only while the Anchor resolves.

**Architecture:** pnpm monorepo with a Hono+Postgres API (domain rules live in the API), shared Zod types, and an Expo mobile app (map + auth + upload) plus a thin native Marker-AR module (ARKit Image Tracking / ARCore Augmented Images). Drafts soft-hold Density Cap with TTL; World Save requires a resolvable Anchor (ADR-0002); MVP Anchors are Marker-first (ADR-0001).

**Tech Stack:** TypeScript, pnpm workspaces, Turborepo, Hono, Drizzle ORM, Postgres, Better Auth, S3-compatible object storage (MinIO locally), Zod, Expo (dev client), expo-router, react-native-maps, Jest (API), Maestro or Detox later for E2E (not in MVP unit tasks).

## Global Constraints

- Domain vocabulary from root `CONTEXT.md` only (Asset, Draft, Placement, MapPosition, Pose, Anchor, Marker, World, Author, Viewer, Nearby, Cluster, Density Cap, Degraded, Safety Review, Report, Takedown, Attribution, Co-creation).
- Follow ADR-0001 (Marker-first) and ADR-0002 (World Save requires Anchor).
- MVP Visibility is World only (no Friends/Link/Event).
- `DENSITY_CAP_PER_ANCHOR = 10`
- `DRAFT_TTL_HOURS = 24`
- `NEARBY_RADIUS_METERS = 500`
- `NEARBY_LIMIT = 50`
- `CLUSTER_RADIUS_METERS = 25`
- Attribution defaults to `showAuthorName: true`
- Co-creation defaults to `false` on new Markers
- Pose is immutable after World Save; Asset replace and Attribution toggle are allowed
- Anonymous Viewer may call Nearby and view AR; create Draft / Marker / Placement requires signed-in Author
- Safety Review: pluggable port; local default auto-approves in `NODE_ENV=development`, rejects via admin stub in tests; production wires a real provider later without changing call sites
- No cloud/VPS anchors in MVP
- Package manager: pnpm 9+; Node 22+

---

## File structure (create during tasks)

```
/
├── CONTEXT.md
├── docs/adr/0001-marker-first-anchors.md
├── docs/adr/0002-world-save-requires-anchor.md
├── docs/design/tokens.md           ← FE scaffold (Task 1, docs only)
├── docs/api/mvp.md                 ← BE scaffold (Task 1)
├── docs/db/schema.md               ← BE scaffold (Task 1)
├── package.json                    ← Task 2+
├── pnpm-workspace.yaml
├── turbo.json
├── packages/
│   ├── ui/                         ← later: code tokens from docs/design/tokens.md
│   └── domain/
└── apps/
    ├── api/
    └── mobile/
```

---

### Task 1: Scaffold docs — FE design tokens + BE API/DB design

**Purpose:** Design shelf only — **no application code**. FE = token doc (colors, gaps, radii, type). BE = API contract + DB schema design. Later tasks implement against these docs.

**Files (markdown only):**
- Create: `docs/design/tokens.md`
- Create: `docs/api/mvp.md`
- Create: `docs/db/schema.md`

**Source:** Figma Author Flow (`eYmEd1TdLcmVlHbpocigO7`) / `docs/design/penpot/author-flow/`

**`docs/design/tokens.md` must list:** colors (ink, surfaces, accent, sheet, text roles, danger, map/draft pins), space scale (gaps/padding), radii, typography roles (display/title/body/caption) with sizes — values from Author Flow.

**`docs/db/schema.md` must list:** tables `authors`, `assets`, `markers`, `drafts`, `placements`, `reports`; columns, PK/FK, uniqueness, density soft-hold, Nearby indexes.

**`docs/api/mvp.md` must list:** each MVP route — method/path, auth, request/response, errors (health, auth, assets, drafts, markers, placements, nearby, reports, admin takedown).

- [x] **Step 1: Write `docs/design/tokens.md` from Author Flow**
- [x] **Step 2: Write `docs/db/schema.md`**
- [x] **Step 3: Write `docs/api/mvp.md`**
- [ ] **Step 4: Review with human — no code in this task**

---

### Task 2: `@dtw/domain` + API app skeleton, DB schema, auth

**Files:**
- Create: `packages/domain/**` (constants, geo, Zod schemas + tests) aligned to `docs/api/mvp.md` / `docs/db/schema.md`
- Create: `apps/api/package.json`, `apps/api/tsconfig.json`, `apps/api/drizzle.config.ts`, `apps/api/src/env.ts`, `apps/api/src/db/client.ts`, `apps/api/src/db/schema.ts`, `apps/api/src/auth.ts`, `apps/api/src/index.ts`, `apps/api/src/db/schema.test.ts`, `docker-compose.yml` (postgres + minio)

**Interfaces:**
- Consumes: Task 1 docs (`docs/db/schema.md`, `docs/api/mvp.md`); implements Drizzle to match DB doc
- Produces: `@dtw/domain` exports; Drizzle tables `authors`, `assets`, `markers`, `drafts`, `placements`, `reports`; Better Auth `requireAuthor(c)`; `createApp()` Hono; `GET /health`

```ts
// apps/api/src/db/schema.ts — columns MUST match docs/db/schema.md
```

- [ ] **Step 1: Add docker-compose for Postgres 16 and MinIO**

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: dtw
      POSTGRES_PASSWORD: dtw
      POSTGRES_DB: dtw
    ports: ['5432:5432']
  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: miniopass
    ports: ['9000:9000', '9001:9001']
```

- [ ] **Step 2: Write schema shape test (drizzle table names exist)**

```ts
import { assets, drafts, markers, placements } from './schema';
test('core tables exported', () => {
  expect(assets).toBeDefined();
  expect(drafts).toBeDefined();
  expect(markers).toBeDefined();
  expect(placements).toBeDefined();
});
```

- [ ] **Step 3: Run — expect FAIL, then implement schema + drizzle client + env zod + Better Auth email OTP or magic link + `GET /health`**

- [ ] **Step 4: `docker compose up -d` && migrate && `pnpm --filter @dtw/api test` PASS**

- [ ] **Step 5: Commit**

```bash
git commit -m "feat(api): scaffold Hono app, Postgres schema, and auth"
```

---

### Task 3: Object storage + Asset upload + Safety Review port

**Files:**
- Create: `apps/api/src/storage/s3.ts`, `apps/api/src/safety/port.ts`, `apps/api/src/safety/stub.ts`, `apps/api/src/routes/assets.ts`, `apps/api/src/routes/assets.test.ts`

**Interfaces:**
- Consumes: `requireAuthor`, S3 client
- Produces:

```ts
export type SafetyReviewPort = {
  reviewAsset(input: { assetId: string; storageKey: string; contentType: string }): Promise<'approved' | 'rejected'>;
};

// POST /assets/upload-url  -> { assetId, uploadUrl, storageKey }
// POST /assets/:id/complete -> runs SafetyReviewPort, sets safetyStatus
// GET /assets/:id
```

- [ ] **Step 1: Failing test — complete upload with stub approver sets `approved`**

```ts
test('complete marks asset approved when stub approves', async () => {
  // create author session, request upload-url, complete
  // expect safetyStatus === 'approved'
});

test('complete leaves draftable asset rejected when stub rejects', async () => {
  // inject RejectAllSafety
  // expect safetyStatus === 'rejected'
});
```

- [ ] **Step 2: Implement S3 presign, routes, `StubSafetyReview` (approve unless `SAFE_REVIEW_MODE=reject-all`)**

- [ ] **Step 3: Tests PASS; commit**

```bash
git commit -m "feat(api): asset upload URLs and pluggable safety review"
```

---

### Task 4: Density Cap + Draft soft-hold TTL

**Files:**
- Create: `apps/api/src/domain/density.ts`, `apps/api/src/domain/density.test.ts`, `apps/api/src/domain/drafts.ts`, `apps/api/src/domain/drafts.test.ts`, `apps/api/src/routes/drafts.ts`

**Interfaces:**
- Consumes: `DENSITY_CAP_PER_ANCHOR`, `DRAFT_TTL_HOURS`, drafts/placements tables
- Produces:

```ts
export async function countOccupiedSlots(anchorKey: string, now: Date): Promise<number>;
// occupied = world placements on marker + active drafts soft-holding that marker/map cell

export async function assertCanSoftHold(anchorKey: string, now: Date): Promise<void>;
// throws DensityCapExceededError if count >= DENSITY_CAP_PER_ANCHOR

export async function createDraft(input: {
  authorId: string;
  assetId?: string;
  mapPosition: { lat: number; lng: number };
  softHoldAnchorId?: string;
  now: Date;
}): Promise<Draft>;

export async function expireDueDrafts(now: Date): Promise<number>;
```

API:
- `POST /drafts` (auth) — creates Draft, `expiresAt = now + 24h`, soft-holds if `softHoldAnchorId` provided or hashed geocell of MapPosition
- `GET /drafts/mine` (auth)
- `DELETE /drafts/:id` (auth) — status `discarded`, releases soft-hold
- Cron/interval or lazy: `expireDueDrafts` on write paths

Anchor key for density without marker yet: `geocell:{latBucket}:{lngBucket}` at ~25m resolution until `softHoldAnchorId` set when Author picks existing Marker.

- [ ] **Step 1: Unit tests for cap — 10 placements block 11th soft-hold; expired draft frees slot**

```ts
test('11th active draft on same anchor throws DensityCapExceededError', async () => { /* ... */ });
test('expired draft no longer counts', async () => { /* ... */ });
```

- [ ] **Step 2: Implement density + drafts domain + routes**

- [ ] **Step 3: PASS + commit**

```bash
git commit -m "feat(api): drafts with density soft-hold and TTL"
```

---

### Task 5: Marker generation + Co-creation flag

**Files:**
- Create: `apps/api/src/marker/generateMarkerPng.ts`, `apps/api/src/marker/generateMarkerPng.test.ts`, `apps/api/src/routes/markers.ts`, `apps/api/src/routes/markers.test.ts`

**Interfaces:**
- Consumes: S3, `requireAuthor`
- Produces:

```ts
export function generateMarkerPng(markerId: string): Buffer; // unique high-contrast pattern

// POST /markers { mapPosition } -> creates Marker, uploads PNG, coCreationEnabled=false
// GET /markers/:id -> metadata + imageUrl
// PATCH /markers/:id { coCreationEnabled } -> creator only
// GET /markers/nearby?lat&lng&radiusMeters — optional helper for on-site pick
```

Rules:
- Only creator may Save Placements unless `coCreationEnabled === true`
- Enforce in Save path (Task 6), but set flag here

- [ ] **Step 1: Test PNG non-empty and different markerIds differ in bytes**

- [ ] **Step 2: Implement generator (e.g. `pureimage` or `sharp` + seeded QR-like pattern) + routes**

- [ ] **Step 3: PASS + commit**

```bash
git commit -m "feat(api): generate Marker images and co-creation flag"
```

---

### Task 6: Save Placement (World) — Anchor required

**Files:**
- Create: `apps/api/src/domain/placements.ts`, `apps/api/src/domain/placements.test.ts`, `apps/api/src/routes/placements.ts`

**Interfaces:**
- Consumes: density, drafts, assets safety, markers co-creation, `SavePlacementInputSchema`
- Produces:

```ts
export async function savePlacement(input: {
  authorId: string;
  assetId: string;
  markerId: string;
  mapPosition: { lat: number; lng: number };
  pose: Pose;
  showAuthorName?: boolean; // default true
  draftId?: string;
  now: Date;
}): Promise<Placement>;

// POST /placements  (auth)
// GET /placements/:id  (public if worldVisible && !takenDown)
// PATCH /placements/:id { assetId?, showAuthorName? } — Author only; cannot change pose/map/marker
// DELETE /placements/:id — Author only; sets worldVisible=false or hard delete
```

Invariants (test each):
1. Asset `safetyStatus` must be `approved` else 400
2. Marker must exist; if `authorId !== creator` then `coCreationEnabled` else 403
3. `assertCanSoftHold` / occupied slots including this Save < cap
4. Pose + MapPosition stored once; PATCH rejecting pose/map/marker changes
5. If `draftId` provided: must belong to author, status `active`; then set `consumed`
6. No Save without `markerId` (schema + route)

- [ ] **Step 1: Write failing tests for invariants 1–6**

- [ ] **Step 2: Implement domain + route**

- [ ] **Step 3: PASS + commit**

```bash
git commit -m "feat(api): World Save Placement with Anchor and immutability rules"
```

---

### Task 7: Nearby + Cluster helpers + Report/Takedown

**Files:**
- Create: `apps/api/src/domain/nearby.ts`, `apps/api/src/domain/nearby.test.ts`, `apps/api/src/domain/reports.ts`, `apps/api/src/routes/nearby.ts`, `apps/api/src/routes/reports.ts`, `packages/domain/src/cluster.ts`, `packages/domain/src/cluster.test.ts`

**Interfaces:**
- Produces:

```ts
// GET /nearby?lat=&lng=&radiusMeters?&limit?
// returns World placements within radius, order by distance, limit NEARBY_LIMIT
// excludes takenDown / non-worldVisible; never returns Drafts

export function clusterPlacements(
  items: Array<{ id: string; lat: number; lng: number }>,
  radiusMeters: number,
): Array<{ lat: number; lng: number; placementIds: string[] }>;

// POST /reports { placementId, reason } — Viewer may be anon (reporterAuthorId null) or auth
// POST /admin/placements/:id/takedown — protected by ADMIN_TOKEN header for MVP
```

- [ ] **Step 1: Tests — nearby ordering, draft exclusion, cluster merge within 25m, takedown hides from nearby**

- [ ] **Step 2: Implement + commit**

```bash
git commit -m "feat(api): Nearby query, cluster helper, report and takedown"
```

---

### Task 8: Expo mobile app — auth, map Nearby, Draft create

**Files:**
- Create: `apps/mobile/*` as in file structure (`app/_layout.tsx`, `app/index.tsx`, `app/sign-in.tsx`, `app/draft/new.tsx`, `src/api/client.ts`, `src/map/NearbyMap.tsx`, `src/map/clusterPlacements.ts`)

**Interfaces:**
- Consumes: API `/nearby`, `/drafts`, `/assets/*`, auth session cookie/bearer
- Produces: Viewer map screen; Author sign-in; “New Draft” long-press on map

Behavior:
- `index`: show user location + Nearby pins; tap Cluster → list; tap Placement → detail (preview image + Attribution)
- Anonymous OK for map
- `draft/new`: pick/upload Asset → create Draft at chosen MapPosition (requires auth; redirect sign-in)
- Show Author’s active Drafts as distinct markers (only to Author)

- [ ] **Step 1: `npx create-expo-app@latest apps/mobile -t expo-template-blank-typescript` then add expo-router, react-native-maps, secure store**

- [ ] **Step 2: Implement API client + NearbyMap using `clusterPlacements` from `@dtw/domain`**

- [ ] **Step 3: Manual checklist (document in PR): cold launch map loads pins from local API**

- [ ] **Step 4: Commit**

```bash
git commit -m "feat(mobile): Expo map Nearby, auth gate, and Draft creation"
```

---

### Task 9: Native Marker-AR module + on-site Save flow

**Files:**
- Create: `apps/mobile/modules/marker-ar/` (Expo native module), `apps/mobile/app/marker/[id].tsx`, `apps/mobile/app/ar/[markerId].tsx`, `apps/mobile/src/ar/types.ts`

**Interfaces:**
- Produces RN API:

```ts
// apps/mobile/modules/marker-ar/src/index.ts
export type AnchorPose = {
  position: { x: number; y: number; z: number };
  rotation: { x: number; y: number; z: number; w: number };
};

export type MarkerArViewProps = {
  markerImageUrl: string;
  placements: Array<{ id: string; pose: AnchorPose; imageUrl: string; showAuthorName: boolean; authorDisplayName?: string }>;
  mode: 'place' | 'view';
  onMarkerResolved: () => void;
  onMarkerLost: () => void;
  onPoseCommitted?: (pose: AnchorPose) => void; // place mode when Author confirms
};

export declare const MarkerArView: React.ComponentType<MarkerArViewProps>;
```

Native behavior:
- Register Marker PNG as ARKit reference image / ARCore Augmented Image
- `view`: when resolved, draw Placement Assets at Pose; on lost → Degraded UI (host screen shows map preview only / banner “寻找 Marker”)
- `place`: Author manipulates one Asset plane, confirms → `onPoseCommitted`
- Host screens: download Marker image; call `POST /placements` with pose; handle DensityCap / Safety errors

**Status note:** This task is `ready-for-human` for native iOS/Android wiring; agents may stub `MarkerArView` with a simulator that toggles resolved/lost for UI integration tests.

- [ ] **Step 1: Stub JS module with resolve/lost buttons for Expo Go-less dev client**

- [ ] **Step 2: Wire `ar/[markerId]` Viewer + Author place → Save**

- [ ] **Step 3: Replace stub with ARKit/ARCore image tracking (human)**

- [ ] **Step 4: Commit stub + screens; follow-up commit for native**

```bash
git commit -m "feat(mobile): Marker AR place/view flow and Save Placement"
```

---

### Task 10: Degraded UX, Attribution toggle, replace Asset, Co-creation UI

**Files:**
- Modify: `apps/mobile/app/placement/[id].tsx`, `apps/mobile/app/ar/[markerId].tsx`, `apps/mobile/app/marker/[id].tsx`
- Modify: `apps/api` only if gaps found

Behavior checklist:
- Anchor lost → hide AR Asset meshes; keep Placement detail / map thumbnail (Degraded)
- Author settings: toggle Attribution; replace Asset (re-run Safety Review before visible)
- Marker creator toggle Co-creation
- Error toasts for Density Cap / review rejected (keep Draft, allow replace Asset)

- [ ] **Step 1: Implement UI states + API calls**

- [ ] **Step 2: Manual QA script in `.scratch/mvp/qa-mvp.md`**

- [ ] **Step 3: Commit**

```bash
git commit -m "feat(mobile): Degraded view, attribution, asset replace, co-creation toggles"
```

---

### Task 11: README + local runbook + seed script

**Files:**
- Create: `README.md`, `apps/api/src/scripts/seed.ts`, `.scratch/mvp/qa-mvp.md`

- [ ] **Step 1: Document `docker compose up`, migrate, `pnpm dev`, Expo dev client, env vars**

- [ ] **Step 2: Seed one Author, approved Asset, Marker, Placement in downtown test coords**

- [ ] **Step 3: Commit**

```bash
git commit -m "docs: local runbook and seed data for MVP"
```

---

## Out of scope (explicit)

- Friends / Link / Event Visibility
- Cloud/VPS Anchors
- Production Safety Review vendor wiring (port only)
- Social graph, DMs, likes
- Admin console UI (takedown via token endpoint is enough)
- Cross-platform store submission polish

## Spec coverage self-check

| Domain / decision | Task |
| --- | --- |
| Asset upload + Safety Review | 3, 6, 10 |
| Draft + soft-hold TTL | 4 |
| Marker-first Anchor | 5, 9, ADR-0001 |
| World Save requires Anchor | 6, ADR-0002 |
| Pose/Map immutable | 6 |
| Nearby + limit + Cluster | 7, 8 |
| Density Cap | 4, 6 |
| Degraded | 9, 10 |
| Report / Takedown | 7 |
| Attribution default on | 6, 10 |
| Co-creation default off | 5, 6, 10 |
| Anon Viewer / Auth Author | 2, 8 |

## Execution notes

- Prefer Subagent-Driven execution task-by-task.
- Task 9 native AR may stay stubbed until a human lands ARKit/ARCore; do not block Tasks 1–8, 10–11 on store-ready tracking.
