# DB schema design — MVP

Source of truth for Drizzle in later tasks. Domain language: root `CONTEXT.md`. Constants: `DENSITY_CAP_PER_ANCHOR=10`, `DRAFT_TTL_HOURS=24`, `NEARBY_RADIUS_METERS=500`, `NEARBY_LIMIT=50`, `CLUSTER_RADIUS_METERS=25`.

## Conventions

- IDs: UUID v4 (or UUIDv7), stored as `uuid`
- Timestamps: `timestamptz`, UTC
- MapPosition: `lat` / `lng` as `double precision` (WGS84)
- Pose: JSONB `{ position: {x,y,z}, rotation: {x,y,z,w} }` relative to Marker Anchor
- Soft deletes: Prefer status / `worldVisible` / `takenDownAt` over hard delete unless noted

## ER (logical)

```
authors 1──* assets
authors 1──* drafts
authors 1──* markers          (creator)
authors 1──* placements
assets  1──* drafts           (nullable while drafting)
assets  1──* placements
markers 1──* placements
markers 1──* drafts           (optional softHoldAnchorId)
placements 1──* reports
```

## Tables

### `authors`

| Column | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | Same as auth user id when wired |
| `display_name` | text not null | Attribution default source |
| `created_at` | timestamptz not null | default now() |

Better Auth session/user tables may sit alongside; `authors` is the domain profile.

### `assets`

| Column | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | |
| `author_id` | uuid not null FK → authors | |
| `storage_key` | text not null | S3 object key |
| `content_type` | text not null | e.g. image/png |
| `byte_size` | int not null | |
| `safety_status` | text not null | `pending` \| `approved` \| `rejected` |
| `created_at` | timestamptz not null | |

Indexes: `(author_id)`, `(safety_status)`.

### `markers`

| Column | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | |
| `creator_author_id` | uuid not null FK → authors | |
| `image_storage_key` | text not null | Generated Marker PNG |
| `pattern_hash` | text not null | Dedup / integrity of pattern |
| `co_creation_enabled` | boolean not null | default **false** |
| `map_position_lat` | double precision not null | Where Marker lives on map |
| `map_position_lng` | double precision not null | |
| `created_at` | timestamptz not null | |

Indexes: `(creator_author_id)`, `(map_position_lat, map_position_lng)`.

### `drafts`

Author-only; never in Nearby. Soft-holds Density Cap until expire / discard / consume.

| Column | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | |
| `author_id` | uuid not null FK → authors | |
| `asset_id` | uuid null FK → assets | May attach later |
| `lat` | double precision not null | Draft MapPosition |
| `lng` | double precision not null | |
| `soft_hold_anchor_id` | uuid null FK → markers | If holding a known Marker |
| `soft_hold_key` | text not null | `marker:{id}` or `geocell:{latBucket}:{lngBucket}` (~25m) |
| `expires_at` | timestamptz not null | `created_at + 24h` |
| `status` | text not null | `active` \| `expired` \| `consumed` \| `discarded` |
| `created_at` | timestamptz not null | |

Indexes: `(author_id, status)`, `(soft_hold_key, status)`, `(expires_at)` where status = active.

**Density:** Occupied slot = World Placement on Marker **or** `active` Draft with same `soft_hold_key`. Cap = 10 per key.

### `placements`

World unit after Save. Pose + MapPosition + Marker immutable after insert.

| Column | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | |
| `author_id` | uuid not null FK → authors | |
| `asset_id` | uuid not null FK → assets | Must be `approved` at Save |
| `marker_id` | uuid not null FK → markers | Required (ADR-0002) |
| `lat` | double precision not null | Immutable |
| `lng` | double precision not null | Immutable |
| `pose` | jsonb not null | Immutable |
| `show_author_name` | boolean not null | default **true** |
| `world_visible` | boolean not null | default true; Author delete → false |
| `taken_down_at` | timestamptz null | Platform Takedown |
| `created_at` | timestamptz not null | |

Indexes:

- `(marker_id)` — density count
- `(lat, lng)` — Nearby bounding box prefilter
- `(world_visible, taken_down_at)` — visibility filter
- `(author_id)`

Nearby candidate set: `world_visible = true` AND `taken_down_at IS NULL`, then haversine filter ≤ 500m, order by distance, limit 50. Never include drafts.

### `reports`

| Column | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | |
| `placement_id` | uuid not null FK → placements | |
| `reporter_author_id` | uuid null FK → authors | null = anonymous Viewer |
| `reason` | text not null | |
| `created_at` | timestamptz not null | |
| `resolved_at` | timestamptz null | |

Index: `(placement_id)`, `(resolved_at)`.

## Invariants (enforce in domain layer)

1. World Save requires resolvable Marker / Anchor id (stored as `marker_id`).
2. Asset `safety_status` must be `approved` before Placement enters World.
3. Density: placements on marker + active drafts on same `soft_hold_key` < 10.
4. Co-creation: if `author_id ≠ markers.creator_author_id`, require `co_creation_enabled`.
5. PATCH Placement may change `asset_id` / `show_author_name` only — not pose, map, or marker.
6. Consuming a Draft sets `status = consumed` and releases soft-hold.
