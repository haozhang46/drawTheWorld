# World Save requires a resolvable Anchor

A Placement only enters World when Save succeeds with a resolvable Anchor. Map-only or remote intent is a Draft (Author-only, soft-holds Density Cap with TTL), never a Placement. Viewers therefore never discover points that cannot, in principle, be shown in AR at marker-grade when the Marker is present.

## Considered Options

- **Anchor required for World Save (chosen)**: keeps Nearby honest; Degraded is only for later resolve failure, not for never-anchored World spam.
- **Allow World Save with MapPosition only**: fills the map quickly, but most camera opens would be Degraded and Density Cap becomes meaningless for AR.
- **Publish Drafts as World “unanchored” pins**: same discovery pollution; blurs Draft vs Placement in the domain model.
