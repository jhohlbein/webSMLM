# experimental_data

Drop real SMLM stacks here for benchmarking and validation (Phase 2 onward).

## These files are not committed

Everything with an image extension is git-ignored — see [`.gitignore`](.gitignore).
That is deliberate:

- **This repository is public.** Committing a stack publishes it, along with any
  licensing or prior-publication implications.
- **GitHub rejects files over 100 MB** and warns above 50 MB. Raw SMLM stacks are
  routinely far larger.
- **Git history is permanent.** A large binary committed once bloats every future
  clone even if it is deleted in a later commit.

The files are still fully usable locally — they are simply untracked.

## If a fixture *should* be committed

A small, deliberately-cropped stack (a few MB, e.g. 50 frames of 128×128) makes a
genuinely useful regression fixture. Force-add it and say where it came from:

```bash
git add -f experimental_data/small_crop.tif
```

For anything larger, prefer a download link or a Zenodo deposit referenced here
over committing the bytes.

## Current benchmark dataset

`GATTA-PAINT-80R-raw_cropped.tif` — GATTAquant GATTA-PAINT 80R DNA-PAINT
nanoruler (80 nm mark-to-mark), acquired on a Leica GSD system, cropped from the
`80nm 80ms top image.lif` series and concatenated by ImageJ 1.54r.

| Property | Value |
|---|---|
| Frames | 1999 |
| Frame size | 82 × 83 px (cropped from 180 × 180) |
| Bit depth | 16-bit, uncompressed |
| Byte order | **big-endian (MM)** — ImageJ wrote the stack BE; the per-frame originals are LE |
| **Pixel size** | **99.2 nm** (from XResolution 4294967295/42605 = 100808.996 px/cm) |
| Exposure | 80 ms |
| On disk | 26.5 MB |
| Decoded Float32 working set | 51.9 MB — fits the default 3 GB budget, so it loads fully into RAM |

Two reasons this file is a good fixture beyond raw speed:

- **It exercises the big-endian 16-bit decode path**, which was previously
  reasoned-about but never runtime-verified.
- **It carries a resolution ground truth** — the 80 nm mark-to-mark spacing lets
  Phase 4 (FRC) be validated against a known distance rather than only
  self-consistency.

## Useful properties to note for benchmarking

When adding a stack, record these — they determine which Phase 2 optimizations
matter and make timings comparable:

| Property | Why it matters |
|---|---|
| Frame size (px) | Band-pass cost scales with pixel count |
| Frame count | Determines whether streaming or in-RAM loading is used |
| Bit depth / endianness | Exercises the TIFF decode paths |
| Pixel size (nm) | Needed for correct nm-space output |
| Approx. σ_PSF (px) | Sets the DoG kernel size — the dominant cost term |
| Emitter density | Affects detection count and the fit-vs-detect time split |
