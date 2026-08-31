# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla JavaScript Tetris. No build step, no package manager, no dependencies — just `index.html`, `style.css`, and `game.js`.

## Running / testing

There is no build, lint, or test tooling in this repo. To try changes:

```bash
open index.html          # macOS, opens directly in the browser
# or
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no automated tests. Verify changes by playing the game in a browser (check movement, rotation/wall-kicks, line clears, scoring, level speed-up, pause, and game-over/restart).

## Architecture

Everything lives in `game.js` (~300 lines), driven by DOM elements defined in `index.html` and styled in `style.css`.

- **Board model**: `board` is a `ROWS × COLS` matrix (`createBoard`); each cell is `0` (empty) or a 1–7 color index.
- **Pieces**: `PIECES` holds the 7 tetrominoes as square matrices. `rotateCW` rotates via transpose + row-reverse.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells.
- **Wall kicks**: `tryRotate` retries the rotated shape at `ox ± 1/2` before giving up.
- **Game loop**: `loop(ts)` runs on `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and advances the piece once `dropAccum >= dropInterval`.
- **Line clears**: `clearLines` scans bottom-up, removes full rows, unshifts empty rows at the top, and scores via `LINE_SCORES[n] * level`.
- **Level/speed**: level increments every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)`.
- **Ghost piece**: `ghostY` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2` in `draw()`.
- **Rendering**: `draw()` redraws grid + locked board + ghost + current piece on the `#board` canvas each frame; `drawNext()` renders the preview piece on `#next-canvas`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece immediately collides; shows the `#overlay` with a restart button wired in `init()`.

### Control flow

```
init() → createBoard() → next = randomPiece() → spawn() → requestAnimationFrame(loop)
loop(ts): accumulate dt → drop piece or lockPiece() when dt ≥ dropInterval → draw() → recurse
keydown: move / rotate (tryRotate) / softDrop / hardDrop / togglePause
```

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If you change `COLS`/`ROWS`/`BLOCK`, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
