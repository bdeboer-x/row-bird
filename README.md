# Row Bird

A slow-paced, Flappy Bird–style game controlled by a Concept2 rower (PM5 monitor) over Bluetooth.

The whole game is `index.html`: no server, no libraries, no data collection. It runs entirely in your browser.

## Play

Open the GitHub Pages URL in **Chrome on Android**, wake the PM5, tap **Connect PM5**, pick it from the list, and start.
Holding the phone sideways gives you more warning before each obstacle.

Every stroke is a flap. The bird hops as your drive starts, then gets its main lift when the drive ends, sized by that stroke's power.
Set your **target power** in the menu: a stroke at target power at about 24 strokes a minute keeps the bird level, harder strokes climb, and lighter ones sink.

### Modes

- **Endless:** three lives; see how far you get.
- **Timed:** a 5, 10, 15 or 20 minute run. You can't die; a crash just costs you that pipe's point. The end screen shows your strokes, average watts and crashes.

### Power-ups (toggle in the menu)

Items float between pipes. Fly through them to collect.

- Good (green ring): 🛡️ shield blocks one crash · ❤️ extra life (Endless) / ⭐ +3 points (Timed) · 🪶 lighter for 12s · ↕️ next 4 pipes open wider
- Bad (red ring): ⚓ heavier for 12s, so you need to pull harder · ⛈️ next 4 pipes narrow

**Try without rower** lets you play by pressing and holding the screen for each stroke. Holding about half a second is a target-power stroke, and longer is harder.

## Host on GitHub Pages

1. Create a new public repository and upload `index.html` (and this README).
2. Go to **Settings → Pages**, set **Source** to *Deploy from a branch*, and choose `main` / `(root)`.
3. After a minute, the game is live at `https://<your-username>.github.io/<repo-name>/`.

Web Bluetooth only works from an `https://` address, which GitHub Pages provides.
