# MVP Anchors are Marker-first

World Placements need marker-grade reappearance when an Anchor resolves. For MVP we establish Anchors primarily with a platform-generated visual Marker; environment/cloud anchors are an enhancement later, not a launch dependency.

## Considered Options

- **Marker-first (chosen)**: controllable fidelity, works without a geospatial vendor, matches the honest rule that Degraded view applies when the Anchor does not resolve.
- **Cloud/VPS-first**: better “no sticker” UX, but coverage, cost, and lock-in are high—especially outdoors—and would delay MVP.
- **Session-only Pose**: trivial to ship, but contradicts the marker-grade promise and makes cross-visit AR unreliable by design.
