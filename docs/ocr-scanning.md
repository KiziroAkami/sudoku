# TODO: OCR board scanning

Status: **not started**, deliberately parked. Design notes from the session of
2026-08-26 so none of the reasoning has to be redone.

## The goal

From "Enter my own" on a blank board, upload or photograph a sudoku and have the
81 cells filled in automatically, then land in setup mode for review before
playing.

## Pipeline

Five steps. Only step 2 is genuinely hard.

1. **Get the image.** `<input type="file" accept="image/*">` gives camera or photo
   library on iOS. No permission prompt for the library. Trivial.
2. **Find the board and correct perspective.** See the fork below.
3. **Split into 81 cells.** Trivial once step 2 is done.
4. **Recognise each cell** as blank or 1-9.
5. **Validate, flag low-confidence cells, and let the user review** in setup mode
   before pressing Start playing.

## The fork: screenshots vs photos of paper

**Screenshots of another sudoku app** are the easy case: axis aligned, clean,
consistent glyph rendering, no shadows. Step 2 reduces to finding the strong grid
lines, or even assuming a pre-cropped board. High accuracy is achievable.

**Photos of a newspaper** are a different problem: angle, page curvature, shadows,
uneven light. Needs corner detection, a perspective warp and adaptive
thresholding. Two routes:

- `opencv.js` does all of it and costs about **8 MB**, which is roughly 45x the
  current app size. Rejected on size unless there is no alternative.
- **Hand-rolled homography**, about 150 lines of maths and no dependency. Finicky
  to tune, but testable by generating synthetically warped boards and checking
  they come back straight. This is the preferred route.

**Recommendation: build screenshots first.** It gets most of the value for a
fraction of the difficulty and produces a working pipeline to extend.

## Recognition: do NOT use Tesseract

Tesseract.js is the obvious reach and the wrong tool here. It is built for lines
of prose, it is mediocre on isolated characters in boxes, and its trained data is
megabytes.

**Preferred: a purpose-built digit classifier.** Ten classes (blank plus 1-9), one
narrow job.

- Training data can be **generated synthetically**: render digits in many fonts,
  sizes, blurs and noise levels. Standard approach for this exact problem, needs
  no external dataset.
- Result is a few hundred KB of weights plus about 100 lines of matrix maths to
  run. No framework needed for a net this small.
- Roughly 20x smaller than Tesseract with better accuracy on this task.
- Prerequisite: a one-time offline training script. Python with numpy and Pillow
  is already working on the dev machine (see the venv bootstrap note).

**Fallback for screenshots only:** plain template matching. Digits within one
source are pixel consistent, so normalise each cell and correlate against
templates for 1-9. Effective for clean screenshots, near useless for photos.

## What de-risks the whole feature

**The app already has a solver, so it can check the OCR's work.** A scan that
produces a board with no solution, or with many solutions, is almost certainly a
misread. Combined with dropping the user into setup mode to review, with
low-confidence cells flagged, the bar drops from "OCR must be perfect" to "OCR
must save typing". Build it this way regardless of the recognition approach.

## Size budget

The app is about 183 KB today, and **143 KB of that is the embedded fonts**. The
actual logic is around 40 KB. A 400 KB classifier takes the app to roughly 600 KB:
still small, still instant, still fully offline. It is Tesseract or opencv.js that
would break the self-contained property, not the feature itself.

## Open decisions

1. Screenshots only to begin with, or straight at photos of paper?
2. Is roughly 600 KB acceptable for the finished app?

## Related

The explained-hint feature (see the hint engine in `index.html`) shares nothing
with this except the validation step, so the two are independent.
