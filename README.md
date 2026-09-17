# SolarArcade

A single self-contained `index.html` — no build, no server. Open it (or host it as a static page) and it works.

## How it's built

- `index.html` is the arcade hub: name/company entry, game menu, combined leaderboard.
- Each game (Solar Convoy, Panel Balance, Load Tetris) is a full standalone HTML file, base64-encoded inside a `<script type="text/plain" id="src-...">` block and loaded into an `<iframe>` when played.
- Hub ↔ game communication is `postMessage` only: the game asks the hub to `submitScore` / `getLeaderboard` / `goToMenu`.
- Scores are stored via the Claude `db` capability when available, falling back to `localStorage` on the visitor's device otherwise (see `localModeNote` in the hub).

## Booth workflow

1. Visitor types their name (+ optional company) once in the hub — it's carried into whichever game they pick.
2. They pick a game from the menu and play in the iframe; "Back to arcade" returns to the menu.
3. Every game always includes Aruna AI as a fixed opponent, so every run is comparable and feeds one shared "Arcade Champions" leaderboard, ranked by how close the visitor got to the AI.
4. Load Tetris and Panel Balance both support an optional second human player (left player = A/D, right player = arrow keys). Leave the second name blank and that player's panel is hidden — solo vs AI. Load Tetris also has a "Hide Aruna AI" checkbox for a pure two-human match.

## Editing a game's source

The embedded games are base64, not readable HTML, in the file. To edit one:

```python
import re, base64
html = open('index.html').read()
m = re.search(r'<script type="text/plain" id="src-loadtetris">(.*?)</script>', html, re.S)
open('loadtetris.html', 'wb').write(base64.b64decode(m.group(1)))
```

Edit `loadtetris.html`, then re-encode it back into the same block:

```python
new_b64 = base64.b64encode(open('loadtetris.html','rb').read()).decode()
html = html[:m.start(1)] + new_b64 + html[m.end(1):]
open('index.html', 'w').write(html)
```

Swap `loadtetris` for `solarconvoy` or `panelbalance` for the other games.
