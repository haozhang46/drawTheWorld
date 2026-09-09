# Design tokens — Author Flow

Source: [Figma Author Flow](https://www.figma.com/design/eYmEd1TdLcmVlHbpocigO7) · local refs in `docs/design/penpot/author-flow/`.  
Phone frame: **390 × 844**. Later `@dtw/ui` / mobile styles must match this doc — do not invent a second palette.

## Colors

| Token | Hex | Role |
| --- | --- | --- |
| `ink` | `#1C1F1A` | App chrome / primary CTA fill / text on sheet |
| `surfaceDark` | `#2A2F28` | Inner dark surface |
| `cameraWell` | `#3D4538` | Camera / AR viewport well |
| `track` | `#4A5344` | Progress track, borders on dark |
| `textOnDark` | `#E8E6DF` | Primary text on dark |
| `textMutedOnDark` | `#B8C4A8` | Secondary text on dark |
| `accent` | `#C8F07A` | Lime accent — CTA label, guides, progress fill, map pin |
| `sheet` | `#F4F1EA` | Bottom sheet / light panel |
| `textOnSheet` | `#1C1F1A` | Titles on sheet (same as ink) |
| `textMutedOnSheet` | `#5C6356` | Body on sheet |
| `textFaintOnSheet` | `#7A8270` | Hints on sheet |
| `danger` | `#C45C4A` | Errors / Safety rejected (not in frame; reserved) |
| `mapPin` | `#C8F07A` | World Placement pin (= accent) |
| `draftPin` | `#B8C4A8` | Author-only Draft pin (= muted on dark) |
| `border` | `#4A5344` | Default border on dark (= track) |

Accent wash used in crop guide: `accent` at **8%** opacity.

## Space (gap / padding)

| Step | px | Typical use |
| --- | --- | --- |
| `0` | 0 | — |
| `1` | 4 | Tight chip gaps |
| `2` | 8 | Icon–label |
| `3` | 12 | Screen inset (frame chrome), small stack |
| `4` | 16 | Control padding, status row |
| `5` | 24 | Sheet content inset, section gap |
| `6` | 32 | Large section / sheet top padding |
| `7` | 48 | Rare hero spacing |

CTA height in frames: **48**. Eligibility bar height: **44**.

## Radii

| Token | px | Typical use |
| --- | --- | --- |
| `sm` | 8 | Crop guide |
| `md` | 12 | Chips / eligibility pill |
| `lg` | 14 | Primary CTA |
| `xl` | 20 | Camera well |
| `2xl` | 28 | Sheet top corners |
| `3xl` | 32 | Inner phone surface |
| `phone` | 40 | Device frame (mock only) |

## Typography

Sans UI (Author Flow uses Inter in Figma; mobile may map to the app’s loaded UI font — keep **sizes/weights**).

| Role | Size | Weight | Use |
| --- | --- | --- | --- |
| `display` | 20 | 700 | Sheet titles |
| `title` | 15 | 600 | Primary CTA label |
| `body` | 13 | 400 / 600 | Body, status |
| `caption` | 12 | 400 | Hints, secondary |

Line-height: ~1.25–1.4 of size; prefer +4–8px over size for multi-line body.
