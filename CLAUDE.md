# Hilti Quest

Single-file pixel-art presentation (`index.html`, canvas 384x216, no dependencies). A hero walks a road
through five stops; each stop opens a card. Controls: Space next, B back, Enter reveal mystery stop / reopen card, F fullscreen, R restart.

All presenter-editable content lives in the `CONFIG` block at the top of the script. ALL card text is in one array,
`CONFIG.stops`; each stop has exactly four text fields: `title`, `boss` (`""` = plain card, no battle), `line` (one short
line: the fight line on a boss card, may use `\n` for several lines on a plain card) and `loot` (what the boss drops;
shown on the intro and victory cards). Keep every line under ~12 words. The fixed card words ("BOSS BATTLE", "VICTORY!",
"ACQUIRED", "TEAMMATES JOINED", prompts) live in `CONFIG.ui`. Non-text game setup sits beside the text in each stop:
`reward` (items in drop order; an entry may be `{ gear, note }` where the note is a card line shown just before that item
drops), `art`, `team`, `sprite`, `weapon`, `mystery`, `isCastle`. Item names come from `CONFIG.gearNames`.

## Stop order and items picked up

| # | Stop (card title)                       | `art`    | Landmark                       | Items picked up                                   |
|---|-----------------------------------------|----------|--------------------------------|---------------------------------------------------|
| 1 | HIGH SCHOOL                             | `school` | closed schoolhouse, bleachers  | none (hero starts bare)                           |
| 2 | UNDERGRAD (D1 GOLF) / TEXAS TECH        | `campus` | red + black hall, Double T     | golf club, time management clock, community heart (+ 2 teammates) |
| 3 | BOOTH MIF                               | `hall`   | gothic maroon hall + skyline   | smarty glasses (beating Finals), leadership shield (leadership development certificate) |
| 4 | MASKED RIDER CAPITAL                    | `office` | glass tower with logo sign     | Excel badge (the weapon that beats THE ROLL-UP TWISTER) |
| 5 | HILTI                                   | (castle) | castle + crane, hat on the gate| hard hat                                          |

All 7 items: golf club, time management clock, community heart, smarty glasses, leadership shield, Excel badge, hard hat
(config keys `club`, `clock`, `heart`, `glasses`, `shield`, `excel`, `hat`). Inventory tray (`drawHud`, top-left, 7 compact 12x14 slots, no label) order = `GEAR_KEYS`; slots stay dimmed silhouettes until the item is earned.

- Items on a battle stop are granted only once the boss is beaten; on other stops, on arrival.
- Worn on the hero (a female character): golf club (hand), leadership shield (arm), smarty glasses (face), hard hat (head, last).
  Clock, heart and Excel badge are inventory-only.
- Undergrad (D1 golf): beating the boss grants club + clock + heart, and two teammates join and follow the hero
  (both female: one blonde, one ginger; `MATE_STYLES`).
- Booth MIF: the community heart (earned at Undergrad) powers the fight (line "COMMUNITY POWERS THE FIGHT."; loot "SMARTY GLASSES AND A LEADERSHIP SHIELD."): the hero and
  both teammates stand beside each other during the Finals fight. Winning drops the glasses (they appear on his face + inventory), then the card line
  "LEADERSHIP DEVELOPMENT CERTIFICATE" shows and the shield drops (on his arm + inventory). The boss is FINALS (`reward` entries can be
  `{ gear, note }`; the note row appears ~0.9s before that item drops, see `dropSchedule`).
- Masked Rider Capital: boss THE ROLL-UP TWISTER (small tornado, `sprite: "tornado"`, drawn at scale 3). Card: "WEAPON: EXCEL." and loot
  "FINANCIAL MODELING AND A DEEP READ ON PRIVATE MARKETS." (the loot box wraps to two lines). The hero holds the Excel badge up,
  then throws it at the boss during the fight (`weapon: "excel"`); on the win it drops into the inventory (inventory-only item).
  The party (hero + two teammates) stands beside him.
- Copy status: High School, Masked Rider Capital and Hilti lines, plus boss names and lines, are placeholder.

## Landmark details (each stop looks different)

- High School (`drawSchool`): a closed gate across the entrance with a red CLOSED sign + padlock, empty aluminium bleachers (with one
  forgotten water bottle), and a faded, sagging GRADUATION banner under the eaves; dark windows, nobody home.
- Texas Tech (`drawCampus`): red and black brick hall with a black roof, a Double T plaque on the clock tower and above the door, a red
  raider flag (masked-rider face emblem, `RAIDER`), black/red golf flag on the putting green.
- Booth MIF (`drawHall`): gothic maroon stone hall (lancet windows, rose window, buttress pinnacles, belfry, crocketed spire), a Chicago
  skyline behind it (Willis Tower with twin antennas, tapered Hancock, other towers), and a stack of exam papers with a red A+ and a pencil.
- Masked Rider Capital (`drawOffice`): modern glass curtain-wall tower with a rooftop sign showing the masked-rider logo (`RIDER_LOGO`, on a gold
  badge) next to "MRC", plus an annex and a glass lobby.
- Hilti (`drawCastle`): the red-roofed castle, now with a yellow tower crane behind it (a steel beam hangs over the keep) and a red hard hat
  hung on the gate.
- `LAND` footprints (`l`/`r`/`h`) and the castle block in `staticBlocks()` were widened for the bleachers, raider flag, skyline and crane;
  retune them if you change a building's size, since sign placement and scenery avoid those boxes.

## Mystery reveal (High School -> Undergrad)

- High School: the hero has no gear. Card text: "LEVEL 1. / JUNIOR AND SENIOR YEAR: CANCELED. / GOAL: DIVISION I GOLF. / WHERE?"
  (`line` may hold several lines separated by `\n`).
- The Undergrad stop carries `mystery: { label: "TEXAS TECH", title: "TEXAS TECH (D1 GOLF)" }`. Until revealed it is drawn as a
  flat grey silhouette of its landmark with a big "?", the map label and "NEXT:" line read "???", and the map shows "ENTER: REVEAL".
- Enter on the map (when the next stop is a hidden mystery) reveals it: short white flash + sparkles, the landmark turns to
  full colour and the label becomes TEXAS TECH. Arriving at the stop also reveals it, so walking on never skips it.
  Once revealed, Enter goes back to reopening the previous stop's card. The card header uses `mystery.title` after reveal.
- Implementation: `isMystery/labelOf/titleOf/revealStop`; the silhouette is the landmark drawn on an offscreen canvas
  (`ctx` is temporarily swapped to `silCtx`) and filled grey with `source-in`.

## Battle cards (Undergrad, Booth MIF, Masked Rider Capital)

A stop with a non-empty `boss` is a battle card (the engine builds `st.battle` from the config fields); `team: N` adds followers.
- Card flow: intro (boss title, the stop's `line`, loot line) -> Space = FIGHT (short boss-hit animation,
  Space skips it) -> VICTORY summary. The stop's card shows the boss until it is won; Enter reopens the VICTORY view.
- Items and followers appear only after the win (`won[i]`). Leaving the card without fighting and pressing Space on the
  map reopens it, so an unfinished fight blocks the road.
- Every battle card draws the party (`drawParty`): the hero with the items worn so far, plus teammates beside him.
  The party hops during the fight; on victory each reward drops onto the hero in turn (`drops`, 0.4s apart).
- Undergrad boss: THE CALENDAR DRAGON ("FOUR-TIME LETTER WINNER."; loot "TIME MANAGEMENT, COMMUNITY."), `sprite: "dragon"`.
- Boss sprites: `SPR_BOSS_DRAGON` (`"dragon"`, calendar page on its belly), `SPR_BOSS` (angry golf ball, `"ball"`, unused) and
  `SPR_BOSS_EXAM` (angry exam paper, `"exam"`, Booth). 
- Teammates trail the hero along the road (`mates()`, `drawMate`), and disappear if the hero walks back before the stop.

## Resolution and art detail

- The canvas is 768x432 (`CW`x`CH`); game logic, layout and text still use a 384x216 grid (`W`x`H`) and `ctx` carries a 2x
  transform (`SS`), so every existing coordinate is unchanged. `fit()` scales the canvas by whole device pixels
  (`image-rendering: pixelated`) so it stays crisp, letterboxed when the window is not an exact multiple.
- Detailed art is drawn on a half-unit grid: `hi(x, y, fn)` runs `fn` with origin (x, y) and 0.5 scale, so `rect()` there
  paints single device pixels. The hero (`drawHeroHi`, 16x26 grid), teammates (`drawMateHi`, 12x18) and all buildings
  (`drawSchool/Campus/Hall/Office/Castle`) use it; shapes are `[x, y, w, h, colour]` boxes (`shapes`) or PAL-letter rows (`pix`).
- The hero is a female character (long brown hair, blue jacket, navy pleated skirt, white sneakers) on a 24x40 hi-px grid; the two
  teammates (`MATE_STYLES`) are 20x32: a blonde in a green sleeveless dress with a ponytail, and a ginger in a purple hoodie and jeans.
  All three have eyes, brows, blush and mouth, and an idle bob (`idleBob`: body bob, head lag, hair sway). On the map the hero is
  12x20 grid units, teammates 10x16; on cards the party is drawn at 2x that.
- Worn gear is drawn on the hero in `drawHeroHi`: golf club gripped in the right hand with the iron head up over her shoulder
  (`CLUB_SHAPES`), glasses on her face, the leadership shield on her arm (`SHIELD_L`), hard hat on her head last. Inventory icons,
  bosses, scenery and the font are still the older 1-unit pixel art. The mystery silhouette uses an offscreen canvas at the same 2x size.

## Code notes

- Stops are spaced evenly along the road (`t = (i+1)/count`), so reordering or adding stops moves them.
  Landmark offsets (`dx`/`dy`) and footprints (`l`/`r`/`h`) are per-`art` in the `LAND` table; retune them if a
  landmark moves to a different position on the road.
- Scenery (trees, props, flowers) is generated with a seeded RNG and avoids the road, landmarks, castle hill and pond.
- Stop labels (and START) are small dark wooden signs with a light border (`drawSigns`), drawn last so nothing covers them.
  `layoutSigns` picks, per label, the nearest free spot that avoids the road (plus the head-room the hero and teammates
  occupy above it), the hero at every stop, each landmark and its status badge, the castle, the pond, the HUD (gear tray,
  progress pips, controls bar, NEXT/ENTER text) and the other signs; a dotted line links a sign back to its stop. It reruns
  only when a label changes (the mystery reveal). If a landmark `LAND` footprint or the HUD moves, update `staticBlocks()`.
- Status badges (green check = done, grey padlock = locked, bobbing arrow = next) are small tabs attached to the top of each stop's
  sign, drawn in `drawSigns`; the landmark itself no longer carries a floating badge. Each sign reserves 11 units above it for the tab,
  labels over 14 chars wrap to two lines, and `layoutSigns` tries every placement order (`perm`) and keeps the shortest total
  sign-to-stop distance. Long Masked Rider / Texas Tech signs can sit 50-110 units from their stop; a dotted line links them.
- The controls hint is a two-line bar drawn on the canvas at the top right (`drawHintBar`, `HINT_BAR`), clear of START/HIGH SCHOOL.
- Card headers drop to the small font when the title is too wide for the big one.
