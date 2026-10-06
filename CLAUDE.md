# Hilti Quest

Single-file pixel-art presentation (`index.html`, canvas 384x216, no dependencies). A hero walks a road
through five stops; each stop opens a card. Controls: Space next, B back, Enter reveal mystery stop / reopen card, F fullscreen, R restart.

All presenter-editable content lives in the `CONFIG` block at the top of the script. ALL card text is in one array,
`CONFIG.stops`; each stop has exactly four text fields: `title`, `boss` (`""` = plain card, no battle), `line` (one short
line: the fight line on a boss card, may use `\n` for several lines on a plain card) and `toolkit` (the skills the tools
stand for, shown on the card after "ADDED TO TOOLKIT:" once the boss is beaten). Keep every line under ~12 words. The fixed card
words ("BOSS BATTLE", "VICTORY!", "ADDED TO TOOLKIT", "TEAMMATES JOINED", "TOOLKIT", prompts) live in `CONFIG.ui`. Non-text game setup
sits beside the text in each stop: `tools` (the tools added to the toolkit at that stop, in drop order; an entry may be
`{ tool, note }` where the note is a card line shown just before that tool drops), `art`, `team`, `sprite`, `weapon`, `mystery`,
`isCastle`. Tool names come from `CONFIG.toolNames`.

## Stop order and tools picked up

| # | Stop (card title)                       | `art`    | Landmark                       | Tools added to the toolkit                        |
|---|-----------------------------------------|----------|--------------------------------|---------------------------------------------------|
| 1 | HIGH SCHOOL                             | `school` | closed schoolhouse             | none (hero starts bare)                           |
| 2 | UNDERGRAD (D1 GOLF) / TEXAS TECH        | `campus` | red + black hall, Double T     | golf club, time management clock, community heart (+ 2 teammates) |
| 3 | BOOTH MIF                               | `hall`   | gothic maroon hall + skyline   | smarty glasses (beating Finals), leadership shield (leadership development certificate) |
| 4 | MASKED RIDER CAPITAL                    | `office` | glass tower with logo sign     | Excel badge (the weapon that beats THE ROLL-UP TWISTER) |
| 5 | HILTI                                   | (castle) | castle + crane, hat on the gate| hard hat                                          |

All 7 tools: golf club, time management clock, community heart, smarty glasses, leadership shield, Excel badge, hard hat
(config keys `club`, `clock`, `heart`, `glasses`, `shield`, `excel`, `hat`). Each tool stands for a skill and goes in the **toolkit bar**
(`drawHud`, top-left, labelled "TOOLKIT", 7 compact 12x14 slots, order = `TOOL_KEYS`); slots stay dimmed silhouettes until the tool
is earned.

Skills the tools stand for (the card text after "ADDED TO TOOLKIT:"): Undergrad = time management, community (club, clock, heart);
Booth = analytical thinking, leadership (glasses, shield); Masked Rider Capital = financial modeling, private markets (Excel badge);
Hilti = the hard hat (dropped by the castle finale, below).

- Tools on a battle stop are added only once the boss is beaten; on other stops, on arrival.
- Worn on the hero (a female character): golf club (hand), leadership shield (arm), smarty glasses (face), hard hat (head, last).
  Clock, heart and Excel badge live only in the toolkit bar.
- Undergrad (D1 golf): beating the boss grants club + clock + heart, and two teammates join and follow the hero
  (both female: one blonde, one ginger; `MATE_STYLES`).
- Booth MIF (`party: "beside"`, `power: "heart"`): the community heart (earned at Undergrad) powers the fight (line "COMMUNITY POWERS THE FIGHT."; toolkit "ANALYTICAL THINKING, LEADERSHIP."): the hero and
  both teammates stand side by side (not staggered behind her) during the Finals fight, and the heart's slot in the toolkit bar glows and
  pulses (red halo + gold frame, `drawToolkitBar`) for the whole fight; on every attack a heart streams from that slot down to the hero
  before the thrown heart leaves her hand (reduced motion: a steady glow, no streaming). Winning drops the glasses (they appear on her face + in the toolkit bar), then the card line
  "LEADERSHIP DEVELOPMENT CERTIFICATE" shows and the shield drops (on her arm + in the toolkit bar). The boss is FINALS (`tools` entries can be
  `{ tool, note }`; the note appears ~0.9s before that tool drops, see `dropSchedule`).
- Masked Rider Capital: boss THE ROLL-UP TWISTER (small tornado, `sprite: "tornado"`, drawn at scale 3). Card: "WEAPON: EXCEL." and toolkit
  "FINANCIAL MODELING, PRIVATE MARKETS." (the text box wraps it to three rows). The hero holds the Excel badge up,
  then throws it at the boss during the fight (`weapon: "excel"`, thrown at the boss on each attack); on the win it drops into the toolkit bar (toolkit-bar-only tool).
  The party (hero + two teammates) stands beside him.
- Copy status: High School, Masked Rider Capital and Hilti lines, plus boss names and lines, are placeholder.

## Landmark details (each stop looks different)

- High School (`drawSchool`, the map landmark): a closed gate across the entrance with a red CLOSED sign + padlock, and a faded, sagging GRADUATION
  banner under the eaves; dark windows, nobody home. (There are no bleachers anywhere now.)
- Texas Tech (`drawCampus`): red and black brick hall with a black roof, a Double T plaque on the clock tower and above the door, a red
  raider flag (masked-rider face emblem, `RAIDER`), black/red golf flag on the putting green.
- Booth MIF (`drawHall`): gothic maroon stone hall (lancet windows, rose window, buttress pinnacles, belfry, crocketed spire), a Chicago
  skyline behind it (Willis Tower with twin antennas, tapered Hancock, other towers), and a stack of exam papers with a red A+ and a pencil.
- Masked Rider Capital (`drawOffice`): modern glass curtain-wall tower with a rooftop sign showing the masked-rider logo (`RIDER_LOGO`, on a gold
  badge) next to "MRC", plus an annex and a glass lobby.
- Hilti (`drawCastle`): the red-roofed castle, now with a yellow tower crane behind it (a steel beam hangs over the keep) and a red hard hat
  hung on the gate.
- `LAND` footprints (`l`/`r`/`h`) and the castle block in `staticBlocks()` were widened for the raider flag, skyline and crane;
  retune them if you change a building's size, since sign placement and scenery avoid those boxes.

## Keys and prompts (every prompt names the one key that works right then)

| Key | Does |
|---|---|
| **Space** (or click) | next: start, walk to the next stop, open its card, close a card, finish; on a boss card it is one attack (pressing mid-swing finishes that swing and swings again) |
| **Enter** | reveal / use: reveal the hidden next stop on the map, and on the scripted High School card use the hand sanitizer and then reveal the answer. Does nothing on other cards |
| **B** | back (close a card, walk back a stop, leave the finish screen) |
| **F** / **R** | fullscreen / restart |

Prompt wording lives in `CONFIG.ui` (`promptSpace` = "PRESS SPACE", `promptEnter` = "PRESS ENTER", plus the High School lines `sanitize`, `ask`, `reveal`).
- Title: "PRESS SPACE TO BEGIN". Map: "PRESS SPACE" always, and "PRESS ENTER TO REVEAL" in place of "NEXT: ???" while the next stop is hidden.
- Cards: the tab on the text box edge says "PRESS SPACE" (boss cards, plain cards, and the High School card once the answer is shown) or
  "PRESS ENTER" (the High School card while the germ is alive or the question is open). The High School card is exclusive: Space does nothing
  until the answer is revealed, and Enter does nothing afterwards, so the prompt is always the only key that works. Enter on any other card is ignored.
- The controls bar (top right, `drawHintBar`) reads: SPACE: NEXT / ATTACK, ENTER: REVEAL / USE, B BACK  F FULL  R RESTART.
- `advance()` is Space, `enterKey()` / `reopen()` are Enter (on the map, Enter without a hidden stop reopens the previous card; this is not advertised).

## Mystery reveal (High School -> Undergrad)

- High School is a scripted boss card (`script: "germ"`, `sprite: "germ"`, `weapon: "sanitizer"`, `hits: 1`). Its popup text is a clean
  list of short lines: "JUNIOR AND SENIOR YEAR:" / "CANCELED BY COVID-19." / "GOAL: PLAY DIVISION I GOLF." (each configured line gets its own
  row), plus a red prompt row. The hero has no tools here, and the toolkit must still be empty after this stop.
  1. THE GERM (`drawGermBoss`: a spiky green germ with ball-tipped spikes, red eyes, angry brows and jagged teeth, with a health bar):
     prompt "PRESS ENTER TO USE HAND SANITIZER." Enter equips a hand sanitizer bottle in her hand and attacks: a spray of droplets
     (`drawSanitizerProp`) hits the germ, the bar empties and the germ fades (one hit, `hitsNeeded()`).
     The bottle is a temporary prop drawn only while the attack and fade play; it is not in `TOOL_ICON`/`TOOL_KEYS` and never enters `earned()`.
  2. The question "WHERE DO I GO TO SCHOOL?" with a grey silhouette + "?" and the prompt "PRESS ENTER TO REVEAL." Enter reveals the
     Texas Tech logo with a short flash and sparkles, and the text "TEXAS TECH." (`card.stage` 0 -> 1).
     The logo is `texas_tech_logo.jpeg` (must sit next to `index.html`; set by `mystery.logo` in CONFIG). `logoCanvas()` pixelates it by drawing
     it onto a small offscreen canvas (56 px wide before cropping in the popup, 16 on the map sign), then removes the white background in code:
     near-white pixels (all channels > 225, a little slack for JPEG noise) connected to the image edge become transparent, white inside the logo
     stays, and the canvas is cropped to the logo. It is scaled up with `imageSmoothingEnabled = false`. The "?" silhouette is that same shape
     filled grey. Reading pixels needs a same-origin image: from `file://` Chrome blocks it, so the logo then falls back to an un-keyed plaque
     (white background kept); serve the folder over http (or open with `--allow-file-access-from-files`) for the transparent version.
     If the image fails to load, the drawn Double T (`drawTTEmblem`) is used. The revealed Texas Tech sign on the overview map shows the same logo, small, left of the name.
  3. Space then closes the card. The scripted card is exclusive: Enter acts (use sanitizer, reveal) and Space only continues once the answer is shown.
  The map's school has a Texas flag on its flagpole (`texasFlag`): a blue bar with a white star on the left, white over red on the right,
  rippling gently (frozen under reduced motion).
- The Undergrad stop carries `mystery: { label: "TEXAS TECH", title: "TEXAS TECH (D1 GOLF)" }`. Until revealed it is drawn as a
  flat grey silhouette of its landmark with a big "?", the map label and "NEXT:" line read "???", and the map shows "ENTER: REVEAL".
- Enter on the map (when the next stop is a hidden mystery) reveals it: short white flash + sparkles, the landmark turns to
  full colour and the label becomes TEXAS TECH. Arriving at the stop also reveals it, so walking on never skips it.
  Once revealed, Enter goes back to reopening the previous stop's card. The card header uses `mystery.title` after reveal.
- Implementation: `isMystery/labelOf/titleOf/revealStop`; the silhouette is the landmark drawn on an offscreen canvas
  (`ctx` is temporarily swapped to `silCtx`) and filled grey with `source-in`.

## The popup is a battle screen (`drawCard`)

Every stop opens the same layout (`CARD`: 352x188 with a 102-high scene, starting at y=26 so it sits below the toolkit bar, which is redrawn bright on top of the dimmed map while a card is open): a slim red title bar (the stop title at 1.5x),
a scene panel for the top two-thirds, and a text box for the bottom third. There is no bullet list and no tool rows. The stop count is a row of
five dots at the bottom of the text box (`dotsRow`: visited = solid, current = larger and red, upcoming = hollow).

**Readable text** (`lightLabel`, the text box): text is dark ink on a light cream panel with a dark border, at 1.5x (`text()` accepts fractional
scales; 1.5 is exactly 3 device pixels, so it stays crisp). That is smaller than the old 2-3x headline text but far higher contrast, and it
reads from across a room. Any text drawn over a backdrop uses the same light panel with a dark border: the boss name + health bar, the
VICTORY! banner, "N TEAMMATES JOINED", the ENTER: REVEAL hint on the map and the Texas Tech label, the map's NEXT / SPACE / ENTER prompts, the controls bar and
the title screen's PRESS SPACE. The wooden stop signs and the toolkit bar keep their own dark styling.
- Characters are drawn large: `drawParty` renders the hero at 4x the map size (48x80 grid units vs 12x20 on the map), standing on the left
  facing right with every earned tool on her (club in hand, glasses, shield on her arm, hard hat at the end). Joined teammates are 40x64,
  standing behind her to the right on slightly higher ground. The boss is drawn at 5x (`BOSS_SCALE`, 60 units; the twister 4x) on the right.
- Scene (`drawScene` -> `SCENES[art]`): every popup has its own backdrop, drawn inside the scene clip and then dimmed (a 36% dark wash) so the
  sprites stand out. The party is at the left, and on boss cards the boss is at the right under a health bar above it (full at the intro,
  draining during the fight). Backdrops:
  - High School (`sceneHallway`): a faded, empty school hallway: lockers, dead ceiling lights, a closed classroom door with a CLOSED notice, a
    faded GRADUATION banner and a stopped clock.
  - Undergrad (`sceneGolf`): a golf course: mowed fairway stripes, a green with a red-and-black flag, a bunker, a pond, pines, red + black tee markers.
  - Booth (`sceneBooth`): the gothic Booth building and a Chicago skyline (Willis Tower, Hancock, a wider lit skyline) at dusk over the lake and a
    stone plaza; `drawHall(x, b, "skyline" | "building")` draws the two halves separately.
  - Masked Rider Capital (`sceneDesert`): a desert at dusk: a sky from dusk purple through warm orange to tan at the horizon, a low sun, red-rock
    mesas (`mesa`) behind layered sand dunes, five saguaro cacti (`saguaro`) and three tumbleweeds (`tumbleweed`) that roll slowly across at
    different depths and speeds (4-9 units/s) with a small hop. They run off `timeNow`, which reduced motion freezes, so they sit still and
    do not hop then. The MRC office tower is not in this backdrop (it is still the map landmark).
  - Hilti (`sceneCastle`, and `sceneFinale` for the finale): sunrise over the castle and its crane.
  Plain stops show the party and any tool tile. See "Mystery reveal" above for the High School germ + Texas Tech sequence.
- Tools (victory): when a boss is defeated each tool is added in three beats (`toolTimes`: pop 1.0s, hold 0.5s, fly 0.6s, 0.6s apart,
  `dropSchedule`). It pops out of where the boss stood, arcs up and floats down into a slot of the toolkit tray in the text box
  (`traySlot`), where it rests under its name (the tool name wrapped to two lines). Then the same icon flies up into the toolkit bar
  (`HUD_BAR` is the shared slot geometry) and the tray slot shows a gold check mark. A tool only counts as earned (lit in the bar, worn
  by the hero) when it arrives, via `toolDone()` in `earned()`. Icons ride on a light backing tile (`backTile`) so dark ones stay readable.
  Plain stops (High School, Hilti) just show a small tool tile in the scene. "N TEAMMATES JOINED" and a big VICTORY! show after a win.
- Text box: before the win, one short line (`line`) at 1.5x, wrapped to <= 3 rows. After a boss win it holds the toolkit tray (one slot
  per tool, tool name under the icon) above the skills line "ADDED TO TOOLKIT: <skills>" (`toolkit`, <= 2 rows), which a tool `note`
  (e.g. LEADERSHIP DEVELOPMENT CERTIFICATE) replaces while that tool is pending. A "SPACE" / "FIGHT" / "ATTACK" tab with a blinking arrow sits
  on the text box edge.

## Battle cards (Undergrad, Booth MIF, Masked Rider Capital)

A stop with a non-empty `boss` is a battle card (the engine builds `st.battle` from the config fields); `team: N` adds followers.
- Fight: Space on a boss card is one attack, and three attacks defeat the boss (`startAttack` / `finishAttack`, timings in `ATK`).
  Each attack is ~0.9s: the hero lunges and uses the stop's tool (`weapon`: the golf club is swung in an arc at Undergrad; the community
  heart (Booth) and Excel badge (Masked Rider Capital) are thrown), the boss goes steadily white and shakes for a moment when the blow
  lands, and the health bar drops a third (green -> amber -> red). Pressing Space mid-swing finishes that attack and swings again, so three
  presses always win. After the third blow the boss fades out (~0.9s), then the VICTORY view appears; Space during the fade skips it.
  The prompt tab reads FIGHT, then ATTACK, then SPACE. The card shows the boss until it is won; Enter reopens the VICTORY view.
- Reduced motion (`prefers-reduced-motion`, or `?reduce-motion` in the URL for testing, `reduceMotion`): the animation clock (`timeNow`)
  is frozen so idle bobbing, orbiting clocks, swaying, clouds and blinking stop; the card appears without its scale-in; there are no
  lunges, thrown/swung tools, shaking, screen flashes, particles or confetti. Attacks resolve fast (`ATK_REDUCED`): a single steady
  white tint (never a strobe), an instant health-bar drop, and a short fade. Tools skip their flights (`toolTimes` shrinks to a quick
  hand-off), so they appear in the tray and then the bar almost immediately. The setting is also picked up live if the OS preference changes.
- Tools and followers appear only after the win (`won[i]`). Leaving the card without fighting and pressing Space on the
  map reopens it, so an unfinished fight blocks the road.
- Every battle card draws the party (`drawParty`): the hero with the tools worn so far, plus teammates beside him.
  The party hops after a win; on victory the tools pop out of the boss, one after another, as described above.
- Undergrad boss: THE 6 AM ALARM (`sprite: "alarm"`; toolkit "TIME MANAGEMENT, COMMUNITY."). Before the fight its text box is a bulleted list:
  heading "I JUGGLED IT ALL:" then four equal bullets (VP OF SAAC, DAILY PRACTICES, 6AM WORKOUTS, RIGOROUS ACADEMICS), all at the same indent and size
  (no sub-bullets). It is written in `line` with `- ` for each bullet; `drawCard` draws a solid square before each and packs five rows closer. Three attacks defeat it; it drops the club, clock and heart, and the two teammates join.
- Bosses are drawn procedurally at about 70 grid units tall, each with its own personality, an idle animation and a health bar +
  name above it (`BOSS_DRAW`; `tinted()` gives the white hit-flash and the grey silhouettes):
  - `alarm` (THE 6 AM ALARM, `drawAlarmBoss`): a giant angry red alarm clock with gold bell ears (they jitter), a hammer, small legs, a face with
    spinning hands, angry eyes and a jagged mouth, and little music notes + ringing lines around it. Idle wobble (x sway + bounce); frozen under
    reduced motion.
  - `exam` (FINALS, `drawFinalsBoss`): a tall stack of exam papers with angry eyes, a jagged mouth, pencil/pen arms, loose sheets
    fluttering off the top and a circled red "A-" on the top sheet.
  - `tornado` (THE ROLL-UP TWISTER, `drawTwisterBoss`): a spinning striped funnel with an angry face and flying spreadsheet cells
    (`sheetCell`) orbiting it, some in front and some behind.
- Teammates trail the hero along the road (`mates()`, `drawMate`), and disappear if the hero walks back before the stop.

## Hilti finale (`finale` in the Hilti stop; all its text is in CONFIG)

The Hilti card is a four-screen sequence (`card.stage` 0-3, `card.st` = seconds in the current screen). Space goes to the next screen; `sceneFinale`
draws the scenes, the text box shows that screen's text. The stop's `line` is the first screen's heading; the rest is in `finale`.
0. **Value towers**: four stone towers (`finale.values`, placeholder names INTEGRITY / COURAGE / TEAMWORK / COMMITMENT) light up one after another
   (windows glow, flags turn gold); each name appears in the text box as its tower lights. (The map castle has two towers; these four exist only in this scene.)
1. **Globe**: a pixel globe (`drawGlobe`, 2x2 blocks, tilted 25 degrees, slowly turning; still under reduced motion) with a red pin per entry of
   `finale.places` (`{ name, lat, lon }`; LUBBOCK, CHICAGO, SCHAAN are placeholders: add yours). Pins pop in one by one and only show on the visible side.
   Continents come from the coarse `LAND_ROWS` table. Text: `finale.globe` (one row per `\n`).
2. **Steps**: three numbered stone steps rise toward the castle on its hill. Text: `finale.steps`, one numbered row each (scale 1 so all three fit).
3. **Hat + toolkit**: the hard hat falls onto her head (`drawParty` drops), then all seven toolkit compartments light up in turn (gold, `drawToolkitBar`),
   then `finale.quote` and `finale.done` ("TOOLKIT COMPLETE.") show. Space during this fast-forwards, then finishes the quest. The hat counts as earned only
   once it lands (`won[i]`), so the toolkit's hat slot is dim until then. Reduced motion: everything appears at once (`finaleTimes`).

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
- Worn tools are drawn on the hero in `drawHeroHi`: golf club gripped in the right hand with the iron head up over her shoulder
  (`CLUB_SHAPES`), glasses on her face, the leadership shield on her arm (`SHIELD_L`), hard hat on her head last. Toolkit icons,
  bosses, scenery and the font are still the older 1-unit pixel art. The mystery silhouette uses an offscreen canvas at the same 2x size.

## Code notes

- Stops are spaced evenly along the road (`t = (i+1)/count`), so reordering or adding stops moves them.
  Landmark offsets (`dx`/`dy`) and footprints (`l`/`r`/`h`) are per-`art` in the `LAND` table; retune them if a
  landmark moves to a different position on the road.
- Scenery (trees, props, flowers) is generated with a seeded RNG and avoids the road, landmarks, castle hill and pond.
- Stop labels (and START) are small dark wooden signs with a light border (`drawSigns`), drawn last so nothing covers them.
  `layoutSigns` puts every name sign directly below its own building (or beside it; above only as a last resort), hugging the building's
  footprint (`LAND` box, the castle block, the START flag). A spot is valid only if it avoids the road (plus the head-room the hero and
  teammates occupy above it), the hero at every stop, every other landmark, the castle, the pond, the HUD (toolkit bar, progress pips,
  controls bar, NEXT / SPACE / ENTER prompts) and the other signs. Candidates come in ranked tiers (hug the building and dodge trees; hug it
  and allow trees; drift up to 90, then 220 units) and the cheapest across tiers wins; `perm` tries every placement order. Any tree or prop
  that would poke through a sign is simply not drawn (`underSign`). There are no connector lines. The layout reruns only when a label
  changes (the mystery reveal). If a landmark `LAND` footprint or the HUD moves, update `staticBlocks()`.
- Status badges (green check = done, grey padlock = locked, bobbing arrow = next) are small tabs attached to the top of each stop's
  sign, drawn in `drawSigns`; the landmark itself no longer carries a floating badge. Each sign reserves 11 units above it for the tab,
  and labels over 14 chars wrap to two lines. The Masked Rider sign sits ~27 units below its tower, beneath the road, because the road runs
  directly under that building and nothing beside it is free.
- Countdown timer (`drawTimer`, `TIMER_BOX`): 8:00 (`TIMER_SECS`) counting down as m:ss in a small dark box in the top-right corner, in the hint bar's empty
  top-right area (its text lines end at x=341, the box starts at 346). It starts on the first Space on the title screen (`timerOn`), runs on real time
  (not frozen by reduced motion), turns gold in the last minute and red at 0:00 (it stops there), pauses on the finish screen, is hidden on the title
  screen and resets with R.
- The controls hint is a two-line bar drawn on the canvas at the top right (`drawHintBar`, `HINT_BAR`), clear of START/HIGH SCHOOL.
