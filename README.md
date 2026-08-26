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
`sudoku-v12`) and push. The version string is the cache key, so an installed
phone keeps serving its old copy until that value changes.

## What is in here

| File | Purpose |
|---|---|
| `index.html` | The whole app: markup, styles, solver and embedded fonts |
| `sw.js` | Service worker. Caches the app so it runs offline |
| `manifest.webmanifest` | Name, icons, colours, and standalone display mode |
| `icons/` | App icons, including the Apple touch icon and a maskable variant |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are |

## Reading a board from a picture

**Scan a picture** in setup mode takes a photo or screenshot of a sudoku, finds
the grid, reads the digits, and drops you into setup so you can look it over
before playing. Cells it was not confident about are outlined in red; editing one
clears its mark. A picture with no grid in it is refused rather than guessed at.

It works on a board that fills the frame or sits inside a screenshot, on light
and dark boards, through highlight bands and jpeg compression, and it treats
pencil marks as an empty cell. Photographs of printed puzzles work too: a
handheld shot is a degree or two off square, which is enough to hide the grid
completely, so the reader finds the angle and straightens the picture first.

Measured at 100% on real phone screenshots and on a photo of a printed puzzle,
and 98.7% per digit on boards rendered in typefaces its template bank has never
seen. Cells it was unsure of are flagged: that threshold is set so it catches
about 79% of misreads while marking 0.3% of correct cells.

## Languages

English and German, switched with the EN/DE toggle at the top right. The choice
is saved, and anything already on screen is rebuilt rather than left behind, so
switching mid-hint keeps you on the same stage.

The hint explanations are generated sentences, not stored strings, so German has
its own phrasing rather than a word-for-word swap. Cell references follow each
language: R4C7 in English, Z4S7 in German.

## Notes on the board

- `1` to `9` places a number. Hold Shift to flip between a number and a note.
- `N` toggles notes mode, `Backspace` erases, `U` undoes, `H` reveals a cell.
- "Enter my own" lets you type any digit into any cell. Nothing is rejected.
  When you start playing it solves the board and tells you honestly whether it
  has one solution, many, or none.
- Paste a puzzle as 81 characters using `0` or `.` for blanks. Line breaks and
  grid borders are ignored, so a copied ASCII grid works.
- **Hint** walks you through the easiest technique that applies, in three stages:
  what to look at, why it narrows to one answer, and the conclusion. The board
  washes out the cells the deduction rules out, marks the digits doing the work
  in green, outlines the row, column or box being reasoned about, and leaves the
  answer as the one cell still untouched. It refuses, with a reason, on a board
  that has no solution or more than one, and it tells you when one of your own
  numbers is wrong rather than pretending a technique applies.
- Every generated puzzle is checked at deal time to be solvable by those
  techniques, so the app can always explain its own boards. A hand-entered board
  may still need something it cannot explain, and it says so.
