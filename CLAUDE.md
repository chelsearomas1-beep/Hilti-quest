# Hilti Quest

**Self-contained:** `index.html` is the whole game: no web fonts (the font is drawn on the canvas), no CDN links, no network requests, and images are base64 data URLs. It works offline from any single copy of the file.

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
| 3 | BOOTH MASTERS IN FINANCE                               | `hall`   | gothic maroon hall + skyline   | smarty glasses (beating Finals), leadership shield (leadership development certificate) |
| 4 | MASKED RIDER CAPITAL                    | `office` | glass tower with logo sign     | Excel badge (the weapon that beats THE ROLL-UP TWISTER) |
| 5 | HILTI                                   | (castle) | castle + crane, hat on the gate| hard hat                                          |

All 7 tools: golf club, time management clock, community heart, smarty glasses, leadership shield, Excel badge, hard hat
(config keys `club`, `clock`, `heart`, `glasses`, `shield`, `excel`, `hat`). Each tool stands for a skill and goes in the **toolkit bar**
(`drawHud`, top-left, labelled "TOOLKIT", 7 compact 12x14 slots, order = `TOOL_KEYS`); slots stay dimmed silhouettes until the tool
is earned.

Skills the tools stand for (the card text after "ADDED TO TOOLKIT:"): Undergrad = golf, time management, community (club, clock, heart);
Booth = analytical thinking, leadership (glasses, shield); Masked Rider Capital = financial modeling, private markets (Excel badge);
Hilti = the hard hat (dropped by the castle finale, below).

- Tools on a battle stop are added only once the boss is beaten; on other stops, on arrival.
- Worn on the hero (a female character): golf club (hand), leadership shield (arm), smarty glasses (face), hard hat (head, last).
  Clock, heart and Excel badge live only in the toolkit bar.
- Undergrad (D1 golf): beating the boss grants club + clock + heart, and two teammates join and follow the hero
  (both female: one blonde, one ginger; `MATE_STYLES`).
- Booth Masters in Finance (`party: "beside"`, `power: "heart"`): the community heart (earned at Undergrad) powers the fight (lines "Community powers the fight." / "Late night study groups with friends" / "Collaboration over competition" / "Let's make each other better"; toolkit "ANALYTICAL THINKING, LEADERSHIP."): the hero and
  both teammates stand side by side (not staggered behind her) during the Finals fight, and the heart's slot in the toolkit bar glows and
  pulses (red halo + gold frame, `drawToolkitBar`) for the whole fight; on every attack a heart streams from that slot down to the hero
  before the thrown heart leaves her hand (reduced motion: a steady glow, no streaming). Winning drops the glasses (they appear on her face + in the toolkit bar), then the card line
  "LEADERSHIP DEVELOPMENT CERTIFICATE" shows and the shield drops (on her arm + in the toolkit bar). The boss is FINALS (`tools` entries can be
  `{ tool, note }`; the note appears ~0.9s before that tool drops, see `dropSchedule`).
- Masked Rider Capital: boss THE ROLL-UP TWISTER (small tornado, `sprite: "tornado"`, drawn at scale 3). Card: "Oh no! A 5-company roll-up acquisition is headed your way." then "Weapon: Excel." and toolkit
  "FINANCIAL MODELING, PRIVATE MARKETS." (the text box wraps it to three rows). The hero holds the Excel badge up,
  then throws it at the boss during the fight (`weapon: "excel"`, thrown at the boss on each attack); on the win it drops into the toolkit bar (toolkit-bar-only tool).
  The party (hero + two teammates) stands beside him.
- Copy status: High School, Masked Rider Capital and Hilti lines, plus boss names and lines, are placeholder.

## Landmark details (each stop looks different)

- High School (`drawSchool`, the map landmark): a closed gate across the entrance with a red CLOSED sign + padlock, and a faded, sagging GRADUATION
  banner under the eaves; dark windows, nobody home. (There are no bleachers anywhere now.)
- Texas Tech (`drawCampus`): red and black brick hall with a black roof, a Double T plaque on the clock tower and above the door, a red
  raider flag (masked-rider face emblem, `RAIDER`), black/red golf flag on the putting green.
- Booth Masters in Finance (`drawHall`): gothic maroon stone hall (lancet windows, rose window, buttress pinnacles, belfry, crocketed spire), a Chicago
  skyline behind it (Willis Tower with twin antennas, tapered Hancock, other towers), and a stack of exam papers with a red A+ and a pencil.
- Masked Rider Capital (`drawOffice`): modern glass curtain-wall tower with a rooftop sign showing the masked-rider logo (`RIDER_LOGO`, on a gold
  badge) next to "MRC", plus an annex and a glass lobby.
- Hilti (`drawCastle`): the red-roofed castle, now with a bold yellow tower crane (dark outline, mirrored so it stands to the left of the keep; kept below y=-112 so the controls bar never hides it; the steel beam on its hook slowly rises and lowers, `lift`; frozen under reduced motion) and a red hard hat
  hung on the gate.
- `LAND` footprints (`l`/`r`/`h`) and the castle block in `staticBlocks()` were widened for the raider flag, skyline and crane;
  retune them if you change a building's size, since sign placement and scenery avoid those boxes.

## Keys and prompts (every prompt names the one key that works right then)

| Key | Does |
|---|---|
| **Space** (or click) | next: start, walk to the next stop, open its card, close a card, finish; on a boss card it is one attack (pressing mid-swing finishes that swing and swings again) |
| **Enter** | reveal / use: reveal the hidden next stop on the map, and on the scripted High School card use the hand sanitizer and then reveal the answer. Does nothing on other cards |
| **Backspace / Left arrow / Page Up** | the old back: close a card, walk back a stop, leave the finish screen |
| **S** / **R** | fullscreen / restart |
| **F** / **B** | presenter shortcuts, not shown on screen: F skips ahead one small step, B goes back one small step (see below) |
| **M** | music on / off (greyed "M MUSIC" in the controls bar while muted) |

Prompt wording lives in `CONFIG.ui` (`promptSpace` = "PRESS SPACE", `promptEnter` = "PRESS ENTER", plus the High School lines `sanitize`, `ask`, `reveal`).
- Title: "PRESS SPACE TO BEGIN". Map: "PRESS SPACE" always, and "PRESS ENTER TO REVEAL" in place of "NEXT: ???" while the next stop is hidden.
- Cards: the tab on the text box edge says "PRESS SPACE" (boss cards, plain cards, and the High School card once the answer is shown) or
  "PRESS ENTER" (the High School card while the germ is alive or the question is open). The High School card is exclusive: Space does nothing
  until the answer is revealed, and Enter does nothing afterwards, so the prompt is always the only key that works. Enter on any other card is ignored.
- The controls bar (top right, `drawHintBar`) reads: SPACE: NEXT / ATTACK, ENTER: REVEAL / USE, R RESET  M MUSIC (the old B BACK / F FULL entries were removed; M greys out while muted).
- `advance()` is Space, `enterKey()` / `reopen()` are Enter (on the map, Enter without a hidden stop reopens the previous card; this is not advertised).

## Mystery reveal (High School -> Undergrad)

- High School is a scripted boss card (`script: "germ"`, `sprite: "germ"`, `weapon: "sanitizer"`, `hits: 1`). Its popup text is a clean
  list of short lines: "JUNIOR AND SENIOR YEAR:" / "CANCELED BY COVID-19." / "GOAL: PLAY DIVISION I GOLF." (each configured line gets its own
  row), plus a red prompt row. The hero has no tools here, and the toolkit must still be empty after this stop.
  1. COVID-19, the germ (`drawGermBoss`: a spiky green germ with ball-tipped spikes, red eyes, angry brows and jagged teeth, with a health bar):
     prompt "PRESS ENTER TO USE HAND SANITIZER." Enter equips a hand sanitizer bottle in her hand and attacks: a spray of droplets
     (`drawSanitizerProp`) hits the germ, the bar empties and the germ fades (one hit, `hitsNeeded()`).
     The bottle is a temporary prop drawn only while the attack and fade play; it is not in `TOOL_ICON`/`TOOL_KEYS` and never enters `earned()`.
  2. The question "WHERE DO I GO TO SCHOOL?" with a grey silhouette + "?" and the prompt "PRESS ENTER TO REVEAL." Enter reveals the
     Texas Tech logo with a short flash and sparkles, and the text "TEXAS TECH.", then four bullets (`mystery.perks`: Team culture, Top 25 women's golf team, Vibrant college town, Full-ride scholarship, in two columns), then the cheer "WRECK EM!" (`CONFIG.ui.wreck`) (`card.stage` 0 -> 1).
     The logo is `texas_tech_logo.jpeg`, embedded in `index.html` as a base64 data URL (the `IMAGES` block at the top; `mystery.logo` is its key; the .jpeg file is kept only as the source). `logoCanvas()` pixelates it by drawing
     it onto a small offscreen canvas (56 px wide before cropping in the popup, 16 on the map sign), then removes the white background in code:
     near-white pixels (all channels > 225, a little slack for JPEG noise) connected to the image edge become transparent, white inside the logo
     stays, and the canvas is cropped to the logo. It is scaled up with `imageSmoothingEnabled = false`. The "?" silhouette is that same shape
     filled grey. Because the image is a data URL, reading its pixels works even from `file://`. If the image fails to load, the drawn Double T (`drawTTEmblem`) is used. The revealed Texas Tech sign on the overview map shows the same logo, small, left of the name.
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
reads from across a room. Any text drawn over a backdrop uses the same light panel with a dark border: the boss name + health bar (small, 1x), the
VICTORY! banner, "N TEAMMATES JOINED", the ENTER: REVEAL hint on the map and the Texas Tech label, the map's NEXT / SPACE / ENTER prompts, the controls bar and
the title screen's PRESS SPACE. The wooden stop signs and the toolkit bar keep their own dark styling.
- Characters are drawn large: `drawParty` renders the hero at 4x the map size (48x80 grid units vs 12x20 on the map), standing on the left
  facing right with every earned tool on her (club in hand, glasses, shield on her arm, hard hat at the end). Joined teammates are 40x64,
  standing behind her to the right on slightly higher ground. The boss is drawn at 5x (`BOSS_SCALE`, 60 units; the twister 4x) on the right.
- Scene (`drawScene` -> `SCENES[art]`): every popup has its own backdrop, drawn inside the scene clip and then dimmed (a 36% dark wash) so the
  sprites stand out. The party is at the left, and on boss cards the boss is at the right under a small health bar panel in the sky to its left, clear of the boss (full at the intro,
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

## Battle cards (Undergrad, Booth Masters in Finance, Masked Rider Capital)

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
- Undergrad boss: THE 5 AM ALARM (`sprite: "alarm"`; toolkit "Golf, time management, community."). Before the fight its text box is a bulleted list:
  heading "I JUGGLED IT ALL:" then four equal bullets (VP of SAAC, Daily practices, 6AM workouts, Rigorous academics, Weekly travel; the boss itself is THE 5 AM ALARM), all at the same indent and size
  (no sub-bullets). It is written in `line` with `- ` for each bullet; `drawCard` draws a solid square before each and packs five rows closer. Three attacks defeat it; it drops the club, clock and heart, and the two teammates join.
- Bosses are drawn procedurally at about 70 grid units tall, each with its own personality, an idle animation and a health bar +
  name above it (`BOSS_DRAW`; `tinted()` gives the white hit-flash and the grey silhouettes):
  - `alarm` (THE 5 AM ALARM, `drawAlarmBoss`): a giant angry red alarm clock with gold bell ears (they jitter), a hammer, small legs, a face with
    spinning hands, angry eyes and a jagged mouth, and little music notes + ringing lines around it. Idle wobble (x sway + bounce); frozen under
    reduced motion.
  - `exam` (FINALS, `drawFinalsBoss`): a tall stack of exam papers with angry eyes, a jagged mouth, pencil/pen arms, loose sheets
    fluttering off the top and a circled red "A-" on the top sheet.
  - `tornado` (THE ROLL-UP TWISTER, `drawTwisterBoss`): a spinning striped funnel with an angry face and flying spreadsheet cells
    (`sheetCell`) orbiting it, some in front and some behind.
- Teammates trail the hero along the road (`mates()`, `drawMate`), and disappear if the hero walks back before the stop.

## Hilti finale (`finale` in the Hilti stop; all its text is in CONFIG)

The Hilti card is a five-screen sequence (`card.stage` 0-4, `card.st` = seconds in the current screen). Space goes to the next screen; `sceneFinale`
draws the scenes, the text box shows that screen's text. The stop's `line` is the first screen's heading; the rest is in `finale`.
0. **Value towers**: four stone towers (`finale.values`, placeholder names INTEGRITY / COURAGE / TEAMWORK / COMMITMENT) light up one after another
   (windows glow, flags turn gold); each name appears in the text box as its tower lights. (The map castle has two towers; these four exist only in this scene.)
1. **Globe**: the castle scene is a stone tower with a glowing arched window and a small spinning globe in it. Prompt: "PRESS ENTER TO ZOOM IN."
   (`finale.zoomIn`; Space does nothing yet). Enter zooms the camera in (0.9s) until the globe fills most of the screen over the dimmed castle
   (`drawGlobeZoom`). The globe (`globeBuf`) is a rotating sphere: blue seas, green land, sand, ice, a light limb/atmosphere glow, slow idle spin until
   the first city. Land comes from a hand-made 128x64 equirectangular mask (`LAND_MASK`, rasterised once from the continent outlines in `CONTINENTS`),
   sampled per pixel into an 80x80 (40x40 in the window) offscreen canvas that is scaled up with smoothing off. Each Space (prompt "PRESS SPACE")
   eases the globe ~1s (smoothstep, shortest way round) to centre the next city of `finale.cities` (`{ name, lat, lon, label }`: London, Shanghai,
   Munich), then a pin drops in with a small pop and a label panel shows its `label`. Pins stay on the globe. Each city also has a framed photo (`photo` in `finale.cities`: the image key in `IMAGES`, the presenter's own `hilti_london.jpeg`, `hilti_shanghai.jpeg`, `hilti_munich.jpeg`, embedded as base64 downscaled to 1400 px; the originals stay in the folder as sources) that pops in beside the globe once the city is reached, on alternating sides, and stays until the next city. The photo is a real <img> element laid over the canvas (`photoEl`, positioned by `updatePhotoOverlay` from `drawCityPics`), so it is smooth and full resolution, not pixelated like the canvas art. To swap one, re-run `sips -Z 1400` on the new file and replace its `IMAGES` entry. After the last city the prompt is
   `finale.cont` ("PRESS ENTER TO CONTINUE."); Enter zooms back out to the castle (text `finale.globe` shows), and Space moves on. Only the working key's
   prompt is shown at each moment. State is `card.g` (`mode` 0 castle / 1 zoomed / 2 back; `updateGlobe`, `startRot`). Reduced motion: no zoom animation,
   no idle spin, no easing (it jumps to each city), no drop or pop.
2. **Steps**: three numbered stone steps rise toward the castle on its hill. Text: `finale.steps`, one numbered row each (scale 1 so all three fit).
3. **Hat + toolkit**: the hard hat falls onto her head (`drawParty` drops), then all seven toolkit compartments light up in turn (gold, `drawToolkitBar`),
   then `finale.quote`, `finale.done` ("Toolkit complete."), `finale.twist` ("Or is it?", after a beat of silence) and `finale.closing` ("Looking forward to a career of building my toolkit with HILTI.", wraps to two rows) type in turn. Space (once the quote has typed and all seven slots have lit) goes on to screen 4, the dark finale. The hat counts as earned only
   once it lands (`won[i]`), so the toolkit's hat slot is dim until then. Reduced motion: everything appears at once (`finaleTimes`).

## Resolution and art detail

- The canvas is 1536x864 (`CW`x`CH`); game logic, layout and text still use a 384x216 grid (`W`x`H`) and `ctx` carries a 4x
  transform (`SS`), so every existing coordinate is unchanged. The pixel art is drawn as blocks of whole grid units, so it stays blocky, while the text is smooth at this resolution. `fit()` fills the window (letterboxed) and may scale by a fraction, which is why the canvas has default smoothing (no `image-rendering: pixelated`).
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
- Stop labels (and START) are small dark wooden signs with a light border and the status badge (check / lock / arrow) inside the sign's left end, hugging their building (`drawSigns`); the Texas Tech logo is on the raider flag in `drawCampus`, not on the sign, drawn last so nothing covers them.
  Every stop's name sign is small (text scale `SIGN_S` 0.5, drawn in `drawSigns` with smooth vector status badges: green check, gold arrow, grey padlock) and hand-placed so it touches its own building, with a little pointer triangle on the edge facing it (`SIGN_AT`: map-unit position per stop index, `x` = sign centre or right edge when `edge: "r"`, `y` = sign top, `point` = up/down/left/right, optional `wrap`): High School just above the schoolhouse roof, Texas Tech just under the hall, Booth against the hall's left wall, Masked Rider Capital (two short lines) against the tower's left side, Hilti under the castle's right tower. Retune `SIGN_AT` if a landmark moves. START (and any stop without an entry) is still placed by the automatic `layoutSigns` search, which now also tries spots on the building's own wall. There are no connector lines. The layout reruns only when a label
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

- Background music (`MUSIC`, `musicStart`, `musicStep`): an upbeat procedural chiptune loop (116 bpm, 8 bars in C: C G Am F C G F G; bright pad, bouncy triangle bass, sparkly arpeggio, a little square-wave tune per bar in `LEAD`, kick + snare + hats) made with Web Audio, so there are no audio files and the game stays self-contained. It starts on the first key press or click (browsers require one), M mutes / unmutes, and it pauses while the tab is hidden.

## Text and the typewriter

- The font is a hand-made proportional 5x7 pixel font with lowercase (`FONT_ART` capitals/digits/punctuation, `LOWER` for a-z: x-height rows 2-6, descenders on rows 7-8), drawn as blocks. One size scale for the whole game (`fs(s)`): callers keep the old steps but they map to the sign look made a little larger: 0.5 map signs, 1 -> 0.75 labels / hint bar, 1.5 -> 1 card body text and prompts, 2 -> 1.25 headlines, 2.5-3.2 -> 2 (the CONGRATULATIONS!! line), 4 -> 3 (the title screen). Body rows are 12 units apart. Each glyph is trimmed to its own width (`FONT`, `ADV`, `textW`). `text()` never upper-cases: card body copy (stop `line`/`toolkit`, finale lines, city labels, tool names, `ui.ask`, `ui.toolkit`) is sentence case; titles, boss names, prompts and signs stay ALL CAPS on purpose. (Two other fonts were tried and dropped: a smooth system sans, and a thresholded Tahoma bitmap.)
- `wrap(str, maxChars)` wraps by pixel width (maxChars x 6 units at scale 1), not character count. Row pitch for 1.5x body text is 14 (13.5 on the victory screen) so descenders never touch the next row; `text()` snaps y to half a unit (1.5x text is exactly 3 device px per font pixel).
- A bullet list on a plain card (Undergrad) is laid out in two equal columns under its heading.
- Typewriter: the text box types out letter by letter (`TYPE`, `typeStart(key)`, `typeEnd()`; deliberately slow so the audience can read and listen: `cps` 25 characters per second, `pause` 0.8 s after every line, `delay` 0.3 s before the first). Each `text()` call is one line that starts when the earlier lines and their pauses are done (`TYPE.cursor`); alignment is measured on the full string; bullet squares appear with their line (`typeReady`). Timed items (the four value names, the final quote and "Toolkit complete") are not paced by the cursor, they appear on their own timers. A text screen retypes only when its key changes (card, stage, victory / not, the victory note), so a fight starting does not retype the same line. Reduced motion: no typing, everything appears at once. Only the card text box types; labels, signs and prompts do not.
- Keys wait for the text: every card's text box tracks `card.tdone` (set in `drawCard` by `typeEnd()`: true once the last line has finished typing; on the Hilti finale also once every tower value has appeared (stage 0) or "Toolkit complete" has appeared (stage 3)). While it is false, Space / Enter / a click do nothing (`advance`, `enterKey`) and the "PRESS SPACE / ENTER" tab on the text box edge is hidden, so nothing can be skipped by accident. B (back) is not gated. Reduced motion has no typing, so keys work immediately.
- The scripted High School card types its question and prompt once: its typewriter key is `r0` while the germ is alive and `r1` afterwards, so the "Where do I go to school?" / "PRESS ENTER TO REVEAL." rows do not retype when the germ finishes fading.

## Tool art and the slow tool drops

- Every tool is drawn by `toolArt(key, ox, oy, k, tint)` on a 24x24 design grid at any scale (club + ball on a tee, red alarm clock, heart, round glasses, red/gold shield with an H, green Excel tile with an X and grid, red hard hat). The same function draws the toolkit bar slots (tiny, `iconAt`), the held / thrown weapon, the tray, the finale hat drop and the end-screen showcase, so the icons are recognisable at every size. The old 7x7 sprites (`SPR_*`, `TOOL_ICON`) are only used by the worn-on-hero art.
- Victory sequence per tool (`toolTimes`, deliberately slow): pops out of the boss and sails to the middle of the scene (0.9 s) -> big showcase, 48 units, on a cream disc with a turning sunburst and its name in a label (2.2 s) -> flies into its tray slot (0.8 s) -> rests there (0.6 s) -> flies into the toolkit bar (0.9 s). The next tool starts 3.4 s after the previous one (`gap`), a noted tool waits 2 s more so its card line can type. The "Added to toolkit: ..." skills line types only after the last tool has landed (`toolsLanded`), and keys / the prompt tab wait until then too (`card.tdone`), so no tool can be skipped. VICTORY! sits top right so it never covers the showcase. Reduced motion: the same steps, but almost instant (`show` 0.6 s).

## Battle sound effects

- `SFX` / `sfxStart` / `battleSfx` (Web Audio, no files, shares the music's master gain so M mutes them too): while a fight is on (battle card, `phase === 1`, from the first attack until the final blow starts the fade) THE 5 AM ALARM plays a bright digital alarm (four quick two-tone square beeps, a rest, repeat) and THE ROLL-UP TWISTER plays wind (looping soft noise through a band-pass that sweeps ~180-1200 Hz with a matching swell, plus a low rumble). Both stop when the boss fades, the card closes, or the game resets. COVID-19 and Finals have no effect.

## The dark finale (Hilti, `card.stage` 4, `drawOutro`, `OUTRO` timings)

Screen 3 (hat + toolkit) now only types the quote. Space then fades the whole screen to black (1 s). At 1.2 s "CONGRATULATIONS!!" (`finale.congrats`) fades in big and gold over a warm glow and twinkling sparkles, with a fanfare (`sfxSting("fanfare")`), confetti and a brief flash; at 2.8 s "Toolkit complete." (`finale.done`) fades in; at 5.6 s "Or is it?" (`finale.twist`) fades in slowly over 1.6 s with a low uneasy drone (`sfxSting("drone")`); at 8.0 s `finale.closing` ("Looking forward to a career of building my toolkit with HILTI.") types out. The "PRESS SPACE" tab and the Space key wait until it has finished; Space then ends the quest. Reduced motion: everything is shown at once (the stings still play) and keys work immediately.

## Objective screen and the logo ending

- After the title, Space opens the objective screen (`S.INTRO`, `drawIntro`, text in `CONFIG.objective`, `.mission`, `.toolkitCount`): the road at night under a dark wash, "Objective:" in gold and "Help Chelsea complete her toolkit." typed out in large letters with a red underline, then the seven toolkit slots fade in as dim silhouettes ("Toolkit: 0 of 7 tools"). The "PRESS SPACE" tab and the Space key wait until all of it has typed and faded in (`introDone`); B goes back to the title. The 8:00 timer starts on the first Space at the title (it is already running here).
- The dark Hilti finale now ends with the Hilti logo instead of the word HILTI: "Looking forward to a career of building my toolkit with" types out, then `hilti_logo.jpeg` (key `finale.logo`, embedded in `IMAGES`, downscaled to 600 px; the .jpeg in the folder is the source) fades in under it, pixelated through `logoCanvas(src, 64)` and drawn with smoothing off at 2.5 units per logo pixel, with a soft red glow and white frame. Keys and the prompt wait until the logo is in (`OUTRO.logoGap`, `logoFade`). If the image fails to load, a drawn "HILTI" is used instead.

## Presenter shortcuts: F (forward) and B (back)

- **F** (`stepForward`) = fast-forward one small step, whatever is waiting. If the screen's text is still typing / animating it completes it at once (that is one step); otherwise it does the one action that moves things along: leaves the title, objective screen or map (the walk is skipped too), attacks once on a boss card, uses the sanitizer / reveals the answer on the High School card, zooms the globe in, visits the next city, zooms out, advances a finale screen, or closes the card. It ignores the "wait for the text" lock, because it completes the text first.
- **B** (`stepBack`) = undo one step. Every Space / Enter / click / F step first saves a snapshot (`act`, `snap`, `HIST`, up to 400) of the game state (screen, stop, card, fight progress, globe, `won`, `revealed`); B restores the most recent one and shows that screen fully played out (typing done, tools landed). Steps that change nothing (a blocked key, a finished-typing F) are not recorded. The timer is never rewound. R clears the history.

## "Important decision" banner (High School)

After the germ is beaten, the screen goes dark and a red glow breathes behind "IMPORTANT" (gold, 0.45 s) and "DECISION" (big, white, 1.0 s, with a short shake), between two glowing gold rails and a bright streak that sweeps across the screen; two heavy low hits play (`sfxSting("decision")`) and the screen flashes (`drawDecision`, `DECISION` timings). It fades out by about 3.3 s and the question "Where do I go to school?" then types (3.7 s), so the text box is blank during the fight and the banner. Keys wait for the question to finish typing; F (skip) and B (back) jump past the banner. Reduced motion: no banner, the question types normally.

## Career-milestone award (Undergrad)

- The Undergrad stop has `award: { title: "CAREER MILESTONE", logo: "the_hartford_logo.svg" }`. After the last tool lands in the toolkit bar (`toolsLanded`) and the skills line has typed, a gold rosette medal with red and blue ribbon tails and a "CAREER MILESTONE" title plate pops up in the middle of the scene on a turning sunburst (3.2 s after the last tool; bell arpeggio via `sfxSting("award")` and a small flash), holds 2.5 s, then shrinks away. Then (`AWARD.logoAt`, 6.2 s) the company logo pops in on a cream plaque where the boss stood. `drawMedal`, `pixelLogo`, `AWARD` timings.
- `the_hartford_logo.svg` (the stag and "The Hartford") is embedded in `IMAGES` as an SVG data URL (the .svg file is the source). `pixelLogo(src, 88)` draws it onto an 88 px wide canvas and thresholds it into hard pixels (dark ink, plus one mid tone for the edges), drawn with smoothing off at 1 unit per pixel, so it is pixelated like the game but the wordmark stays readable. Unlike `logoCanvas`, it works for logos on a transparent background.
- The VICTORY! and TEAMMATES JOINED labels disappear when the award starts (the plaque takes their corner). Keys and the prompt wait until the award sequence is over (`AWARD.done`, 7 s after the last tool). F (skip) jumps to the end of it; B restores the screen with the plaque in place. Reduced motion: no popup, the plaque is simply there.

### More career milestones (Booth and Masked Rider Capital)

- The award mechanism is per stop (`award: { title, logo, color, text }`). Booth Masters in Finance: after Finals is beaten and the tools land, the medal pops up, then `BC_Partners_logo.svg` stands where Finals was and the text box below the tray changes to "Chelsea joins the Private Equity team at BC Partners, analyzing the market for a potential buy-and-build investment". Masked Rider Capital: after THE ROLL-UP TWISTER is beaten and the Excel badge lands, the medal, then `masked_rider_capital_logo.jpeg` where the twister stood, and the text box reads "I ♥ cash flows" (a red heart glyph, `\u2665`, is part of the font). Undergrad (Hartford): "Chelsea joins the Surety Bond team at The Hartford as a Summer Underwriting Analyst".
- `award.text` replaces the "Added to toolkit: ..." line once the logo is placed (it retypes), so the typing, the gating (`AWARD.done`) and F / B behave as for the Undergrad award.
- `pixelLogo(src, wPx, color)`: `color: true` keeps the logo's own colours (BC Partners blue and navy, MRC red and black), drops the white background of JPEGs and the transparent area of SVGs, crops to the logo and thresholds it into hard pixels; without `color` (Hartford) the logo is drawn in one dark ink with one mid tone. Both SVGs and the JPEG are embedded in `IMAGES` (the files in the folder are the sources).

### Beats after the tools (Texas Tech, Booth, Masked Rider Capital)

- After the tools have landed and the skills line has typed, a battle stop can run a chain of **beats** (`beats: [...]` in the config; a stop with only `award` gets `["award"]`). The card waits; each Space (or F) starts the next beat (`card.bt` = the card clock when each began; `beatsOf`, `au` / `bu` / `beatU`), and the next Space is ignored until the current beat has finished (`card.tdone`), exactly like the typing lock. After the last beat, Space closes the card. B restores a screen with the beats started so far fully played out; F skips the rest of the running beat.
- Beat kinds (Undergrad: `award`, `tech10`, `proud`, `next`, `liked`, `grad`, `decide`): `award` (gold medal "CAREER MILESTONE" pops up on a sunburst, then the company logo plaque appears where the boss stood and the text box shows `award.text`; `drawMedal`, `AWARD` timings), `tech10` (the Tech 10 plaque replaces the Hartford plaque and the box shows "Awarded to 10 graduating seniors..." with four bullets: the award is introduced first), `proud` (then, on the next Space, a bigger, more dramatic medal "PROUDEST ACHIEVEMENT": the scene dims, larger sunburst, swelling fanfare; the Tech 10 plaque and its text stay), `next` (the plaque becomes a big grey question mark; box: "... but what next?" with "Grad school?" and "Full time job?"), `liked` (box cleared, "I liked working with brokers and accounts, I enjoyed the financial statement exposure"), `grad` (box: "Let's go to grad school!", "...", "but where?"), `decide` (the second IMPORTANT DECISION banner, `drawDecision(t, "dec2")`; about 3.5 s in the question mark turns into `booth_logo.jpeg`; keys wait about 4.3 s). All text is in `story` on the Undergrad stop.
- Undergrad order: tools -> Space -> CAREER MILESTONE + Hartford -> Space -> Tech 10 plaque + its text -> Space -> PROUDEST ACHIEVEMENT medal -> Space -> "?" + what next -> Space -> liked -> Space -> grad school / but where? -> Space -> IMPORTANT DECISION + Booth logo -> Space closes. Booth and Masked Rider Capital have a single `award` beat.
- The plaque is chosen by `plaqueNow`; the question mark keeps the size of the Tech 10 plaque (`story.proud.w`, `q`). `tech10_logo.svg` (already pixel art, 48x58; drawn 1:1) and `booth_logo.jpeg` are embedded in `IMAGES` (the files in the folder are the sources). `pixelLogo` samples a small SVG at an exact 8x so its pixels stay crisp.
- Reduced motion: each Space still starts the next beat, but a started beat's animations are shown finished.

### Masked Rider Capital opens with its logo (`pre`)

- The Masked Rider Capital stop has `pre: { logo, color, text }`. The card now opens with no boss: the MRC logo (pixelated, on the cream plaque, pops in where the twister will stand) and the line "Chelsea joins the investment team at Masked Rider Capital as a Summer Analyst" typing in the text box (`card.pre`; `openCard`, the `preOn` / `entA` code in the boss block of `drawCard`). Space (once it has typed) clears `pre`: the logo shrinks away and THE ROLL-UP TWISTER comes in (fades and slides in over 0.9 s, `card.entT0`), with its usual "Oh no! A 5-company roll-up acquisition is headed your way." / "Weapon: Excel." lines; the fight and the tools then go as before. After the tools land, the career-milestone medal comes (Space), then the logo again with "I ♥ cash flows" (the existing `award`). Booth and Undergrad are unchanged. F skips the opening line; B restores it.

### Order of the company logo and the career-milestone medal

- At every award stop the **logo comes first, the medal follows**. After the tools land, Space #1 (beat `logo`) puts the company logo plaque where the boss stood and swaps the text box to its line ("Chelsea joins the Surety Bond team at The Hartford ...", the BC Partners line, "I \u2665 cash flows"); Space #2 (beat `medal`) pops up the CAREER MILESTONE medal; then, at Texas Tech, the chain goes on (Tech 10, proudest achievement, ...). Undergrad `beats`: logo, medal, tech10, proud, next, liked, grad, decide. Booth and Masked Rider Capital use the default `["logo", "medal"]`, so the medal is the last thing at the end of those stops. At Masked Rider Capital the stop also opens with the logo before the fight (`pre`).

## Friends at each stop (nobody follows the hero any more)

- The blonde and ginger teammates no longer trail the hero along the road or stand with her at Booth, Masked Rider Capital or Hilti. They only appear on the Undergrad card, once the alarm is beaten (`partyMates()` returns `MATE_STYLES.slice(0, st.team)` there; `mates()` and the map followers were removed; the "2 TEAMMATES JOINED" label is still shown at Undergrad).
- Booth (`friends: ["boothDark", "boothLight"]`) and Masked Rider Capital (`friends: ["mrcGirl", "mrcTall"]`) each have their own two friends, standing beside the hero for the whole stop and gone when she leaves (`FRIENDS`, `drawFriendHi`, used by `drawParty` through `partyMates()`): Booth: two men, one with darker tan skin and black hair (grey sweater), one with lighter skin and brown hair (blue shirt, khakis); MRC: a white woman with long brown hair (red top) and a very tall white man with brown hair (light blue shirt, 40 hi-px tall instead of 32, `tall: 8`, `h: 40`, standing on the ground line so his head stays inside the scene).

## Fullscreen and "meet the characters"

- **S** is fullscreen (it used to be Shift+F; F is now the presenter's skip-ahead key).
- The end screen ("QUEST COMPLETE", "THANK YOU") now has a prompt, "PRESS C TO MEET THE CHARACTERS" (`CONFIG.ui.meetPrompt`). **C** opens three slides (`S.MEET`, `meet`, `MEET_SLIDES`, `drawMeet`) on the night backdrop under the title "MEET THE CHARACTERS" (Space = next slide; after the third, back to the end screen; Backspace / Left arrow = previous slide; B undoes like any step; the timer is paused here):
  1. Texas Tech teammates: the blonde (Kylee) and ginger (Libby) pixel characters beside `libby_kylee` ("Libby & Kylee").
  2. Booth friends: the two Booth pixel men beside `athean` and `rankin` (the darker-skinned black-haired one is labelled Athean, the lighter brown-haired one Rankin).
  3. Masked Rider Capital friends: the pixel woman ("Autumn") and the very tall pixel man ("Trey") beside `trey_autumn`.
- The four photos are embedded in `IMAGES` as `libby_kylee.jpeg`, `athean.jpeg`, `rankin.jpeg`, `trey_autumn.jpeg` (downscaled to about 520-700 px; the originals in the folder are the sources) and drawn smoothly on the canvas. Names and which pixel character goes with which photo are in `MEET_SLIDES`.
