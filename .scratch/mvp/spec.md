# MVP Spec — Draw The World

Source of truth for language: [`CONTEXT.md`](../../CONTEXT.md)  
ADRs: [`0001`](../../docs/adr/0001-marker-first-anchors.md), [`0002`](../../docs/adr/0002-world-save-requires-anchor.md)  
Implementation plan: [`docs/superpowers/plans/2026-09-09-mvp-draw-the-world.md`](../../docs/superpowers/plans/2026-09-09-mvp-draw-the-world.md)

## Product slice

Author: upload Asset → optional Draft on map → on-site Marker Anchor → Safety Review → Save World Placement.  
Viewer (anon OK): Nearby map + Cluster → open camera → see Asset only while Anchor resolves; else Degraded (map preview).

## Non-goals

Directed Visibility, cloud anchors, production moderation vendor UI, social features.
