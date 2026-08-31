# Reading a board from a picture

Status: **shipped** (v11). This file was the design note while it was parked; it
now records what was actually built, what it measures at, and what is still weak.

## What it does

**Scan a picture** in setup mode. Finds the board, reads the digits, lands in
setup so the board can be reviewed before playing. Cells the reader was unsure
about are outlined in red.

Pipeline, in order:

1. Downscale to at most 1600px on the long edge.
2. **Deskew.** Search for the rotation angle, coarse on a downscaled copy then
   fine at full resolution, scored by how well a regular 9-division grid fits.
   Skipped entirely when the picture is already square.
3. **Find the grid.** Collect line candidates two ways (darker than the picture's
   midtone, and darker or lighter than the pixels a few rows either side), then
   FIT a 9-division grid to whatever was found. It locks on from as few as four
   lines, so missing lines are survivable.
4. **Extract ink per cell.** The background is whatever colour dominates that
   cell and ink is whatever differs from it, so black on grey, blue on pale blue
   and white on green all read without knowing the app's palette.
5. **Classify** against a bank of 696 binary templates rendered from 40 font
   faces, by per-pixel disagreement.

## Measured

| | |
|---|---|
| Real phone screenshots (3 images, 117 digits) | 100% |
| Photo of a printed puzzle, handheld, at an angle | 81/81 cells |
| 60 synthetic boards in 12 fonts the bank has never seen | 98.7% per digit |
| Grid found | 60/60 synthetic, 4/4 real |
| Empty cell read as a digit | 0 |
| Template bank | 40 KB |
| Screenshot, no deskew needed | ~190 ms |
| Photo needing deskew | ~2.4 s |

The uncertainty flag fires below a classifier margin of 4. Measured over 2140
digits: wrong reads had a median margin of 2, correct reads 19. That threshold
catches about 79% of misreads while marking 0.3% of correct cells. Raising it to
10 catches every error but flags one cell in eight, which is worse than useless.

## What is still weak

Honest list, in rough order of what would bite first.

- **Only rotation is corrected, not perspective.** A photo taken from a steep
  angle, or of a curved page, still fails. Doing this properly needs a four
  corner detection plus a homography warp, about 150 lines and testable by
  generating synthetically warped boards. That is the obvious next piece.
- **One paper photo in the corpus.** Everything about the print case is tuned
  against a single image. Add more before trusting the numbers.
- **Remaining confusions** on unseen fonts: 8 read as 6, 3 as 1, 4 as 1. Shipping
  all 40 faces (the 98.7% figure withholds 12 of them) helps, but a handful
  survive.
- **Deskew costs about 2.4s** on a photo. Acceptable, not good. The angle search
  re-runs full detection per candidate angle.
- **A picture of graph paper is accepted as a grid** and read as 81 cells. All 81
  come back flagged uncertain, so the safety net fires, but it does not refuse.

## Things that were tried and did not work

Recorded so they are not retried.

- **Dilating the template match** to tolerate stroke weight across typefaces:
  accuracy fell from 97.1% to 82.0% and 6, 8 and 9 collapsed into each other,
  because dilation fills the closed loops that distinguish them.
- **Despeckling** the extracted ink, on the theory that paper texture fragments
  the glyph: cost 1.6 points and fixed nothing. The cell in question turned out
  to contain a perfectly clean digit.
- **Flagging every rejected cell** for review: put 5 marks on a board that read
  perfectly, because outline bleed and real pencil marks are rejected too and are
  confidently not digits.
- **Tesseract.js** was rejected before starting, on size and on being the wrong
  tool for isolated characters in boxes. Nothing since has changed that.

## Non-obvious things worth keeping

- **Do not require all ten grid lines.** Only 8 of 10 survive any threshold that
  separates lines from digits, because highlight bands lift the background under
  the others.
- **Measure a line by how far it reaches, not by its longest unbroken run.** A
  highlight outline drawn across a grid line cuts it in half; it is still a line.
- **A grid line is thin.** Without a thickness cap, a white modal sheet in a
  screenshot registers as one 622px-thick "line" and drags the grid off the board.
- **Pencil marks are separated from digits by interior row gaps**, not by ink
  coverage or aspect. Over 340 digits and 172 note clusters the gap count gave
  perfect separation at every cell size; coverage and aspect both overlapped.
  Count interior gaps only: counting trailing empty rows rejects small cells.
- **Both polarities everywhere.** A dark board draws its grid lines lighter than
  its cells. Do not clamp the thresholds into range either: on a light board the
  midtone IS the white background, and a clamped upper threshold then matches the
  background and scores every row as a line.

Development code, tests and the image corpus live in `.dev-ocr/`, which is
gitignored because the corpus is personal phone photographs.
