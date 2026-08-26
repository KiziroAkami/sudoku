# Sudoku

Sudoku with proper candidate notes, and a board you can fill yourself.

Everything runs on your device. There is no server, no account and no analytics,
and once installed the app makes no network requests at all: the typefaces are
embedded in the page rather than fetched.

## Deploying to GitHub Pages

From this folder:

```
git init -b main
git add .
git commit -m "Sudoku with candidate notes"
gh repo create sudoku --public --source . --push
```

Then in the repository on github.com: Settings, Pages, and under "Build and
deployment" set Source to "Deploy from a branch", branch `main`, folder `/ (root)`.
Save. The first build takes a minute or two.

The address will be:

```
https://<your-username>.github.io/sudoku/
```

## Installing it on the iPhone

1. Open that address in Safari. It must be Safari; Chrome on iOS cannot install
   a home screen app.
2. Tap the Share button, then "Add to Home Screen".
3. Open it from the new icon rather than from Safari. That is what gives you the
   full screen window with no address bar.

Let it finish loading once while you have signal. That first visit is what caches
the app. After it, the icon works with no connection at all.

## Updating it later

Edit `index.html`, then bump `VERSION` in `sw.js` (for example to
`sudoku-v7`) and push. The version string is the cache key, so an installed
phone keeps serving its old copy until that value changes.

## What is in here

| File | Purpose |
|---|---|
| `index.html` | The whole app: markup, styles, solver and embedded fonts |
| `sw.js` | Service worker. Caches the app so it runs offline |
| `manifest.webmanifest` | Name, icons, colours, and standalone display mode |
| `icons/` | App icons, including the Apple touch icon and a maskable variant |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are |

## Notes on the board

- `1` to `9` places a number. Hold Shift to flip between a number and a note.
- `N` toggles notes mode, `Backspace` erases, `U` undoes, `H` reveals a cell.
- "Enter my own" lets you type any digit into any cell. Nothing is rejected.
  When you start playing it solves the board and tells you honestly whether it
  has one solution, many, or none.
- Paste a puzzle as 81 characters using `0` or `.` for blanks. Line breaks and
  grid borders are ignored, so a copied ASCII grid works.
- **Hint** names the easiest technique that applies right now, explains why it
  works, and highlights the cells involved. It refuses, with a reason, on a board
  that has no solution or more than one, and it tells you when one of your own
  numbers is wrong rather than pretending a technique applies.
- Every generated puzzle is checked at deal time to be solvable by those
  techniques, so the app can always explain its own boards. A hand-entered board
  may still need something it cannot explain, and it says so.
