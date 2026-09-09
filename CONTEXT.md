# Draw The World

A mobile AR context where people place images into the real world so others can find them on a map and see them again through the camera. Default visibility is world-shared; directed audiences are a later product surface, not the MVP. A World Placement requires a resolvable Anchor at Save; marker-grade AR appears only while that Anchor resolves. Authors may create an on-map Draft before they are on site to anchor. Drafts soft-hold density with a time limit. One Marker may carry many Placements. Assets need safety review before World. Attribution defaults to showing the Author's name. Others may add Placements to a Marker only when its creator enables co-creation.

## Language

### Core objects

**Asset**:
An image file the user uploads that can be used in one or more placements. A Draft may hold an Asset before review; entering World requires the Asset to pass safety review. If review fails, the Author may replace the Asset on the same Draft and submit again.
_Avoid_: 图片 (alone), upload, media, photo

**Draft**:
An Author-only, not-yet-World record that may hold an Asset and a MapPosition before an Anchor exists. A Draft is not a Placement and is not discoverable in Nearby or World. While alive, a Draft soft-holds capacity under the Density Cap until it expires or is discarded.
_Avoid_: unfinished placement, pending pin, remote drop

**Placement**:
One intentional putting of an Asset into the world at a MapPosition with a Pose and a resolvable Anchor. This is the atomic unit created by Save into World. After Save, its MapPosition and Pose do not change; the Author may replace the Asset or other non-spatial metadata, or delete the Placement. Many Placements may share one Marker/Anchor, still subject to the Density Cap.
_Avoid_: pin (alone), drop, sticker, post, 放入

**MapPosition**:
Where on Earth a Draft or Placement lives, used for map display and (for Placements) nearby discovery.
_Avoid_: location (alone), GPS, coordinates (alone), 地图位置

**Pose**:
The Placement's position and orientation relative to its Anchor. Marker-grade fidelity applies only when that Anchor resolves for the Viewer.
_Avoid_: AR position (alone), transform, 放入图片的位置

**Anchor**:
The persistent AR reference required to Save a Placement. MVP Anchors are established primarily via a visual Marker; environment/cloud anchors are a later enhancement. If a saved Anchor later fails to resolve, the Placement stays on the map but AR fidelity is degraded.
_Avoid_: tracking session, world origin, cloud point (alone)

**Marker**:
A platform-generated visual pattern displayed at the place so devices can establish and later resolve an Anchor. One Marker may back many Placements. By default only the Marker creator's Placements may use it; others need Co-creation enabled.
_Avoid_: QR (as the domain term), sticker, image target (alone)

### Visibility & people

**Visibility**:
Who may see a Placement's content. The default is World. Directed modes (Friends, Link, Event) are planned later and are not part of the MVP.
_Avoid_: privacy, permission, ACL (as product language)

**World**:
The Visibility where any Viewer may discover the Placement on the map and see it through the camera when rules allow.
_Avoid_: public, global, open

**Author**:
The signed-in person who created a Draft or Placement.
_Avoid_: user (when you mean the creator), owner (unless discussing permissions), poster

**Viewer**:
Anyone looking at Placements on the map or through the camera; may be anonymous. Viewers do not see Drafts.
_Avoid_: guest, visitor, user (when you mean the consumer)

**Attribution**:
Whether a Placement shows the Author's display name to Viewers. Defaults to on; the Author may turn it off per Placement.
_Avoid_: byline, credit, signature

**Co-creation**:
A Marker setting, off by default, that allows other Authors to Save Placements onto that Marker's Anchor when the Density Cap allows.
_Avoid_: open wall, shared marker, collaboration (alone)

### Discovery & density

**Nearby**:
The set of World Placements within a fixed distance of the Viewer, returned up to a maximum count, ordered by distance. Drafts are never included.
_Avoid_: feed, search results, local list

**Cluster**:
A map or AR grouping of multiple Placements that share roughly the same place, opened as a list instead of stacking every Asset at once.
_Avoid_: folder, group, stack

**Density Cap**:
The maximum number of Placements allowed on the same MapPosition or Anchor; Save is rejected when the cap would be exceeded. Drafts soft-hold slots for a limited time (TTL), then release if not Saved.
_Avoid_: rate limit (alone), quota (alone)

### Viewing states

**Degraded**:
The Viewer can see map preview for a Placement, but the Asset is not drawn in the camera until the Anchor resolves again.
_Avoid_: failed, offline, approximate AR

### Moderation

**Safety Review**:
The check an Asset must pass before its Placement may enter World. On failure the Draft remains; the Author may replace the Asset and submit again.
_Avoid_: audit, filter (alone), AI check (as domain term)

**Report**:
A Viewer's flag that a Placement may violate rules.
_Avoid_: complaint, flag (alone)

**Takedown**:
Platform removal of a Placement from World visibility after moderation, distinct from the Author deleting their own Placement.
_Avoid_: ban, delete (when the platform acts), hide
