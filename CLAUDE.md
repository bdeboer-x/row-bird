# Row Bird

A slow-paced, Flappy Bird–style game controlled by a **Concept2 rower (PM5 monitor)** over Bluetooth. The player rows; each stroke flaps the bird, and harder strokes fly higher. It is played on an **Android phone in Chrome**, usually held sideways on the rower.

- **Live:** https://bdeboer-x.github.io/row-bird/
- **Repo:** https://github.com/bdeboer-x/row-bird (public, GitHub account `bdeboer-x`, `gh` CLI is logged in on this machine)
- **Deploy:** GitHub Pages serves `main` / root. Pushing to `main` deploys automatically in about a minute (`gh run watch` on the latest `pages-build-deployment` run, then check the live URL). The owner tests on the live site, so push after each round of changes unless told otherwise.

## Files

- `index.html`: the entire game. HTML, CSS and JS in one file with no libraries, no build step and no server. Keep it that way: the owner wanted something small enough to read in full and nothing to abuse when it's hosted publicly.
- `README.md`: player-facing instructions and hosting steps.

## How `index.html` is organised

The script runs top to bottom in these sections:

1. **Tunables:** every gameplay number lives in constants at the top (physics, pipe spacing, item rates, effect durations, the `ITEMS` table). Change feel there first.
2. **Settings:** menu choices (`settings.mode` endless/timed, `durationIdx`, `powerups`), persisted in localStorage.
3. **Rower data:** the `rower` object (spm, pace, last-stroke watts) plus two input sources. Both call the same two hooks, `driveStart()` and `driveEnd(watts)`:
   - **PM5 over Web Bluetooth** (`connectPM5`)
   - **Simulator:** press and hold the screen or space bar as a drive. Holding for 0.5s gives a target-power stroke; longer holds are harder.
4. **Game:** the `game` state object, `update(dt)`, collisions, items, scoring, `endRun`.
5. **Drawing:** canvas rendering.
6. **UI wiring:** menus, HUD, buttons.

### Coordinate system

The canvas is scaled so **1 unit = screen height**, with y pointing down. `worldW` is the width/height ratio. The water line is `GROUND = 0.88`. `birdX` shifts left on narrow (upright) screens so obstacles are visible sooner. **Text is drawn in pixel space** through `drawText()`, because fractional font sizes in world units render unreliably.

### Game states

`menu → countdown → playing ⇄ paused → over`. The 3-2-1-GOOO countdown runs at start and again on resume.

## PM5 Bluetooth details

These follow Concept2's "PM5 Bluetooth Smart Interface Definition". Service UUID pattern: `ce06XXXX-43e5-11e4-916c-0800200c9a66`, rowing service `0030`. All values are little-endian.

| Char | Used for | Bytes read |
|---|---|---|
| `0031` general status | drive start | byte 10 = stroke state; a transition **into 2 (driving)** calls `driveStart()` |
| `0032` additional status | HUD | byte 5 = stroke rate; bytes 7–8 = pace (0.01 s/500m) |
| `0034` sample rate | write `3` = 100ms updates (non-fatal if it fails) | |
| `0036` additional stroke data | per-stroke power | bytes 3–4 = stroke watts; bytes 7–8 = stroke count (used to dedupe) → `driveEnd(watts)` |

- `requestDevice` filters on `namePrefix: 'PM5'`. Notifications are subscribed **one at a time**, because Android dislikes overlapping GATT calls.
- If no power reading arrives within `DRIVE_TIMEOUT` (2s) of a drive start, the lift fires anyway using the last known watts.
- Web Bluetooth requires Chrome (Android/desktop) and **https**. iPhone Safari doesn't support it; Bluefy would be the workaround if that's ever needed.
- **Not verified against real hardware by Claude.** The owner has played it on their PM5 and it works. The exact timing of `0036` relative to the end of the drive is still an assumption.

## Gameplay design (and why)

- **Two-stage flap:** a small hop (`HOP_V`) at drive start so the bird feels connected, then the main lift at drive end, sized **linearly** by `watts / targetWatts` (clamped to `MIN_LIFT`–`MAX_LIFT`). An earlier version used the *previous* stroke's power, which felt disconnected.
- **Balance:** `FLAP_V = 0.085` was derived so that a target-power stroke at 24 spm holds level flight. A higher stroke rate also climbs. If you change `GRAVITY`, `HOP_V` or `FLAP_V`, re-check level flight (see Testing).
- **Target power** (default 120W) is set in the menu. Popups show each stroke's watts, colored: orange <0.8×, white on target, green ≥1.2×, gold ≥1.6×. The bar on the right shows the last stroke against the target line.
- **Glide mode was removed.** It mapped pace to climb rate (and later to height). It felt laggy, because flywheel speed changes over several strokes. Don't reintroduce it without solving that lag.
- **Modes:**
  - **Endless:** 3 lives, max 5.
  - **Timed:** 5/10/15/20 minutes. **You can't die**, because the owner uses it as a workout. A crash forfeits that pipe's point (`pipe.crashed`), followed by 3s of invulnerability. The end screen shows strokes, average watts and crashes.
  - Best scores are stored per mode and length.
- **Power-ups** (menu toggle, on by default) float halfway between pipes, `ITEM_CHANCE = 0.18`. This was halved from 0.35 after the owner found 0.35 too many. 60% good / 40% bad.
  - Good: shield, heart (Endless) / +3 star (Timed, or at max lives), feather (gravity ×0.75 for 12s), wide gaps (next 4 pipes +0.1).
  - Bad: anchor (gravity ×1.25 for 12s, needing ~30% more watts), storm (next 4 pipes −0.07, min 0.22).
  - Pickups are announced with a **large banner across the top** (`showBanner`). The owner asked for this because small text near the bird went unnoticed. Active timed effects also show as chips under the score.
- **Phone held sideways is the intended view.** Upright, you only see about 1.5s ahead. The menu switches to a two-column layout for short landscape screens (`@media (orientation: landscape) and (max-height: 520px)`) so it fits without scrolling.
- The screen is kept awake with the Wake Lock API, and the game auto-pauses when the app goes to the background or the PM5 disconnects.

## localStorage keys (prefix `rowbird.`)

`target`, `runMode`, `durationIdx`, `powerups`, `best` (endless), `best.timed5|10|15|20`. The old keys `mode` and `pace` are leftovers from Glide mode and are unused.

## Testing approach

There's no Node or Chromium on this machine. **Headless Firefox** is used to load test copies of the page and screenshot them:

```bash
firefox --headless --no-remote --profile <scratch>/ffprof --window-size=800,360 --screenshot out.png file://<test>.html
```

- Make a test copy that injects a `<script>` before `</body>`. It can drive the game synchronously (`startGame(); game.state='playing'; loop { driveStart/driveEnd; update(1/60) }`), then freeze it with `update=()=>{}`. Don't freeze it with `lastFrame=Infinity`: that produces a negative dt and NaN physics.
- Stub `beep=()=>{}` and override `performance.now` when simulating time.
- A "steering bot" (power ratio = `1 + (bird.y - nextGap.center) * 4`) is the standard check. It should survive several minutes without crashing.
- A level-flight check: steady target power at 24 spm should keep `bird.y` near 0.45.
- Useful sizes: 800×360 (phone sideways with Chrome's address bar) and 412×860 (upright).
- CSS transitions and animations are captured at t=0. Disable them in the test copy before judging a screenshot. A "switch looks off" false alarm came from this.
- Emoji render grey or faint here (no color emoji font). They're fine on Android.

## Owner preferences

- Plays on an Android phone in Chrome, mounted on the rower, mostly sideways.
- Likes it slow-paced and doable, more workout than twitch game. The 5-minute timed run was described as "a good workout".
- Wants to keep iterating: feel tweaks and new features. Push to Pages after changes so they can test right away.
