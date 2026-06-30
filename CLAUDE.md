# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — the entire game logic lives in a single file.

## Running the game

There is no build/install/test process. Open `index.html` directly, or serve it with any static server:

```bash
python3 -m http.server 8000
# or
npx serve .
```

Then visit `http://localhost:8000`. To verify changes, open the page and play manually (move/rotate/drop pieces, clear lines, trigger pause and game over).

## Architecture

Three files, each with a single responsibility:

- `index.html` — DOM structure: the main `#board` canvas (300×600, 10×20 grid of 30px blocks), a `#next-canvas` preview, the score/lines/level panel, and the pause/game-over overlay.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, organized around a handful of cooperating pieces:
  - **State**: module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class or store. `init()` resets all of them.
  - **Board model**: `board` is a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7`.
  - **Pieces**: the 7 tetrominoes are defined as square matrices in `PIECES`. Rotation (`rotateCW`) transposes + reverses rows rather than using lookup tables; `tryRotate` applies wall kicks by retrying the rotation at x-offsets `[0, -1, 1, -2, 2]`.
  - **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and dropping the piece one row once `dropInterval` is exceeded. Drop speed is `max(100, 1000 - (level - 1) * 90)` ms.
  - **Locking/scoring**: `lockPiece()` → `merge()` writes the piece into `board`, then `clearLines()` removes completed rows (scored via `LINE_SCORES` × level, level = `floor(lines / 10) + 1`), then `spawn()` promotes `next` to `current` and generates a new `next` (triggers `endGame()` if the new piece immediately collides).
  - **Rendering**: `draw()` clears and redraws the grid, locked board, ghost piece (`ghostY()` projects the current piece straight down, drawn at `globalAlpha = 0.2`), and the current piece, all via the Canvas 2D API. `drawNext()` renders the preview canvas separately.
  - **Input**: a single `keydown` listener dispatches on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause).

When tuning gameplay constants (`COLS`, `ROWS`, `BLOCK`, `LINE_SCORES`, initial `dropInterval`), note that changing `COLS`/`ROWS`/`BLOCK` also requires updating the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
