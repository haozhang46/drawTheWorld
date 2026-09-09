# API design — MVP

Implement against this doc. Domain terms: `CONTEXT.md`. DB: `docs/db/schema.md`. Auth: signed-in **Author** vs anonymous **Viewer**.

## Conventions

- Base URL: `/` (Hono app)
- JSON request/response; `Content-Type: application/json` unless noted
- Auth: session cookie or bearer from Better Auth; `requireAuthor` on Author routes
- Admin: header `X-Admin-Token: $ADMIN_TOKEN`
- Errors: `{ "error": { "code": string, "message": string } }`
- IDs: UUID strings

### Error codes

| Code | When |
| --- | --- |
| `UNAUTHORIZED` | Missing/invalid Author session |
| `FORBIDDEN` | Co-creation off, not owner, bad admin token |
| `NOT_FOUND` | Unknown id / not World-visible |
| `VALIDATION` | Zod / bad input |
| `DENSITY_CAP_EXCEEDED` | Soft-hold or Save would exceed cap |
| `SAFETY_NOT_APPROVED` | Asset not approved for World |
| `DRAFT_NOT_ACTIVE` | Draft missing / wrong status / not owner |
| `IMMUTABLE_FIELD` | Attempt to change pose / map / marker |

---

## Health

### `GET /health`

Auth: none  

Response `200`: `{ "ok": true }`

---

## Auth (Better Auth)

Mount Better Auth under `/api/auth/*` (exact paths per library). App needs:

- Sign-in / session for Author
- `GET` session helper used by mobile

Domain profile: ensure `authors` row exists on first successful sign-in (`id`, `display_name`).

---

## Assets

### `POST /assets/upload-url`

Auth: Author  

Body:

```json
{ "contentType": "image/png", "byteSize": 12345 }
```

Response `200`:

```json
{
  "assetId": "…",
  "uploadUrl": "https://…",
  "storageKey": "assets/…"
}
```

Creates Asset with `safety_status: "pending"`. Client PUTs bytes to `uploadUrl`.

### `POST /assets/:id/complete`

Auth: Author (owner)  

Runs Safety Review port; sets `safety_status` to `approved` or `rejected`.

Response `200`: Asset record (see GET).

### `GET /assets/:id`

Auth: Author (owner) for pending/rejected; approved Asset may be read by Author always. Public read of storage via signed URL only when Placement World-visible (implementation detail).

Response `200`:

```json
{
  "id": "…",
  "authorId": "…",
  "contentType": "image/png",
  "byteSize": 12345,
  "safetyStatus": "approved",
  "createdAt": "…"
}
```

---

## Drafts

### `POST /drafts`

Auth: Author  

Body:

```json
{
  "mapPosition": { "lat": 31.23, "lng": 121.47, "accuracyMeters": 10 },
  "assetId": "…",
  "softHoldAnchorId": "…"
}
```

- `assetId` optional  
- `softHoldAnchorId` optional → else `soft_hold_key = geocell:…`  
- `expiresAt = now + 24h`, `status = active`  
- May return `409` / `DENSITY_CAP_EXCEEDED`

Response `201`: Draft.

### `GET /drafts/mine`

Auth: Author  

Response `200`: `{ "items": [ Draft, … ] }` — author’s non-discarded drafts (at least `active`).

### `DELETE /drafts/:id`

Auth: Author (owner)  

Sets `status = discarded`; releases soft-hold.

Response `204` or `200` with Draft.

---

## Markers

### `POST /markers`

Auth: Author  

Body:

```json
{
  "mapPosition": { "lat": 31.23, "lng": 121.47 }
}
```

Server generates Marker PNG → storage; `coCreationEnabled: false`.

Response `201`: Marker (includes image URL).

### `GET /markers/:id`

Auth: none (Viewer OK)  

Response `200`: Marker public fields + `imageUrl`, `coCreationEnabled`, creator display name optional.

### `PATCH /markers/:id`

Auth: Author = creator  

Body: `{ "coCreationEnabled": true }`  

Response `200`: Marker.

---

## Placements

### `POST /placements`

Auth: Author  

Body:

```json
{
  "draftId": "…",
  "assetId": "…",
  "markerId": "…",
  "mapPosition": { "lat": 31.23, "lng": 121.47 },
  "pose": {
    "position": { "x": 0, "y": 0, "z": -0.5 },
    "rotation": { "x": 0, "y": 0, "z": 0, "w": 1 }
  },
  "showAuthorName": true
}
```

Rules:

1. `markerId` required  
2. Asset must be `approved` → else `SAFETY_NOT_APPROVED`  
3. Co-creation / ownership check  
4. Density check  
5. If `draftId`: owner + `active` → set `consumed`  
6. `showAuthorName` defaults true  

Response `201`: Placement.

### `GET /placements/:id`

Auth: none if `worldVisible` and not taken down; else owner or `NOT_FOUND`

Response `200`: Placement + Asset preview URL + Attribution fields when allowed.

### `PATCH /placements/:id`

Auth: Author (owner)  

Body (only): `{ "assetId"?: "…", "showAuthorName"?: boolean }`  

- New `assetId` must be approved (re-review already done on that Asset)  
- Pose / map / marker changes → `IMMUTABLE_FIELD`

### `DELETE /placements/:id`

Auth: Author (owner)  

Sets `worldVisible = false` (Author delete).

---

## Nearby

### `GET /nearby`

Auth: none (Viewer)  

Query: `lat`, `lng` required; `radiusMeters` default 500; `limit` default 50 (max 50).

Response `200`:

```json
{
  "items": [
    {
      "id": "…",
      "lat": 31.23,
      "lng": 121.47,
      "distanceMeters": 42,
      "markerId": "…",
      "showAuthorName": true,
      "authorDisplayName": "…",
      "previewUrl": "…"
    }
  ]
}
```

Excludes Drafts, non-world, taken-down. Ordered by distance.

Clustering is a **client** (or shared domain helper) concern using `CLUSTER_RADIUS_METERS = 25`; API returns flat list.

---

## Reports & Takedown

### `POST /reports`

Auth: optional (anon OK)  

Body: `{ "placementId": "…", "reason": "…" }`  

Response `201`: `{ "id": "…" }`

### `POST /admin/placements/:id/takedown`

Auth: `X-Admin-Token`  

Sets `taken_down_at = now`. Hidden from Nearby / public GET.

Response `200`: Placement id + `takenDownAt`.

---

## Route map (checklist)

| Method | Path | Auth |
| --- | --- | --- |
| GET | `/health` | — |
| * | `/api/auth/*` | Better Auth |
| POST | `/assets/upload-url` | Author |
| POST | `/assets/:id/complete` | Author |
| GET | `/assets/:id` | Author |
| POST | `/drafts` | Author |
| GET | `/drafts/mine` | Author |
| DELETE | `/drafts/:id` | Author |
| POST | `/markers` | Author |
| GET | `/markers/:id` | — |
| PATCH | `/markers/:id` | Creator |
| POST | `/placements` | Author |
| GET | `/placements/:id` | — / owner |
| PATCH | `/placements/:id` | Owner |
| DELETE | `/placements/:id` | Owner |
| GET | `/nearby` | — |
| POST | `/reports` | — / Author |
| POST | `/admin/placements/:id/takedown` | Admin token |
