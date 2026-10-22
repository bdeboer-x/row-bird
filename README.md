# Row Bird

A slow-paced, Flappy Bird–style game controlled by a Concept2 rower (PM5 monitor) over Bluetooth.

The whole game is `index.html`: no server, no libraries, no data collection. It runs entirely in your browser.

## Play

Open the GitHub Pages URL in **Chrome on Android**, wake the PM5, tap **Connect PM5**, pick it from the list, and start.
Holding the phone sideways gives you more warning before each obstacle.

Every stroke is a flap. The bird hops as your drive starts, then gets its main lift when the drive ends, sized by that stroke's power.
Set your **target power** in the menu: a stroke at target power at about 24 strokes a minute keeps the bird level, harder strokes climb, and lighter ones sink.

**Try without rower** lets you play by pressing and holding the screen for each stroke. Holding about half a second is a target-power stroke, and longer is harder.

## Host on GitHub Pages

1. Create a new public repository and upload `index.html` (and this README).
2. Go to **Settings → Pages**, set **Source** to *Deploy from a branch*, and choose `main` / `(root)`.
3. After a minute, the game is live at `https://<your-username>.github.io/<repo-name>/`.

Web Bluetooth only works from an `https://` address, which GitHub Pages provides.
