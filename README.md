# Row Bird

A slow-paced, Flappy Bird–style game controlled by a Concept2 rower (PM5 monitor) over Bluetooth.

The whole game is `index.html`: no server, no libraries, no data collection. It runs entirely in your browser.

## Play

Open the GitHub Pages URL in **Chrome on Android**, wake the PM5, tap **Connect PM5**, pick it from the list, and start.
Holding the phone sideways gives you more warning before each obstacle.

- **Glide mode:** your pace sets your height. Cruise pace (set in the menu) flies mid-screen, faster flies higher, and stopping sinks you.
- **Flap mode:** every stroke is a flap. Harder strokes flap higher.

**Try without rower** lets you play by tapping the screen once per stroke (about 24 taps a minute = cruise).

## Host on GitHub Pages

1. Create a new public repository and upload `index.html` (and this README).
2. Go to **Settings → Pages**, set **Source** to *Deploy from a branch*, and choose `main` / `(root)`.
3. After a minute, the game is live at `https://<your-username>.github.io/<repo-name>/`.

Web Bluetooth only works from an `https://` address, which GitHub Pages provides.
