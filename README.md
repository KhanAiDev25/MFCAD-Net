TEST-PASTE-PROBE# MFCAD-Net

**Turning hand-drawn mechanical sketches into parametric CAD geometry.**

Photograph a sketch on paper. Get back typed geometric primitives, lines, circles and arcs with
real parameters, instead of a picture that merely looks like a drawing.

![Hand-drawn sketch reconstructed as measured geometry](docs/hero_dimensioned.png)

*Left: a real hand-drawn sketch, photographed. Right: reconstructed geometry with automatically
placed dimension labels. Lines in blue, arcs in red, circles in green.*

---

## Why this is hard

A generative model can produce something that looks like a clean drawing. That is not the same as
producing geometry you can machine from. A CAD file needs each entity to be a typed primitive with
parameters: this is a circle at (x, y) of radius r, not a plausible arrangement of dark pixels.

The requirement is exactness, and it rules out most of the obvious approaches.

## Approach

Two stages, deliberately split.

**Stage 1, stroke segmentation.** A two-class U-Net in PyTorch labels every stroke pixel as belonging
to a straight or a curved entity. Trained with a combined Tversky and Dice objective, in FP32:
computing Dice over 512x512 in FP16 overflows to NaN.

**Stage 2, geometric fitting.** Classical geometry, not learned.

| Primitive | Method |
|---|---|
| Lines | Line Segment Detector, then collinear merging |
| Circles | Connected components, then Kasa algebraic fitting |
| Arcs vs circles | Angular coverage threshold, with RANSAC refinement |

**Constraint layer.** Fitted primitives are then made to obey the rules real drawings follow: axis
snapping for hand wobble on near-horizontal and near-vertical lines, concentric grouping,
equal-radius snapping, hub-anchored polar array detection, linear array detection, and fillet
tangency reconstruction solved from the joined lines. Every mutation is validated before it is
applied, with movement caps so a bad fit cannot drag geometry across the part.

## Results

Held-out set of 115 parts.

| Primitive | Metric | F1 |
|---|---|---|
| Lines | instance | **0.94** |
| Circles | coverage | **0.91** |
| Circles | instance | 0.84 |
| Arcs | instance | 0.65 |

![Pipeline output compared against ground truth](docs/comparison_panel.png)

*Input photograph, ground truth, and reconstruction. Arcs remain the hardest class.*

## What did not work, and why

Worth recording, because the failures shaped the architecture.

**End-to-end set prediction.** A DETR-style model predicting primitives directly hits roughly an
8-pixel localisation floor, imposed by the 32x32 feature grid. For CAD output that is unusable.
Splitting into segmentation plus classical fitting removes the floor entirely: least-squares fitting
is limited by stroke width, not by grid resolution.

**HoughCircles.** Produced large numbers of spurious detections on clean masks. Connected components
with Kasa fitting produced zero. That substitution was decisive.

**The original dataset.** Ground truth came from 3D-normalised coordinates overlaid on 2D sketch
images, which is geometrically impossible without camera matrices. The pipeline was rebuilt around
SolidWorks DXF exports with pixel-aligned labels.

**Automatic registration of freehand tracings.** Tested affine at full resolution, coarse-to-fine
multi-start, and per-view homography. Only 8 of 74 sketches reached supervision-grade alignment. A
paired ablation showed the perspective component contributed +1.3 points, indistinguishable from
chance, while initialisation contributed +10.6. The residual distortion is local and non-projective,
so no warp model fixes it. This line is closed.

## Current limitations

Stated plainly, because they matter.

- **Absolute dimensions are not yet valid.** The millimetre labels in the exhibit above are
  uncalibrated. Recovering true scale from an uncalibrated phone photograph requires a known
  reference; the intended solution is a user-entered bounding-box dimension as a single anchor plus
  four-corner perspective rectification.
- **Arcs are the weakest class** at F1 0.65, and the angular-coverage threshold separating arcs from
  full circles is the main tuning lever.
- Evaluated on prismatic mechanical parts. Organic or freeform geometry is out of scope.

## Roadmap

1. **Paper 1 (this work)** sketch to parametric 2D geometry.
2. **Paper 2** 3D and STEP reconstruction from these primitives.
3. **Paper 3** CAM tool-path and G-code generation.

## Demo

A Gradio interface allows upload of a sketch or selection of a sample, with live toggles for axis
snapping, constraint regularisation and tangency reconstruction, and a cached before/after dimension
view that does not re-run inference.

## Context

MS Artificial Intelligence thesis, University of Engineering and Technology (UET) Peshawar,
Department of Electrical Engineering.

The author spent six years programming 8-axis Siemens Sinumerik CNC machines and teaching CNC and CAM
before moving into AI, which is the reason this project treats machinable output, rather than
plausible output, as the bar.

---

Training code and datasets are not published here while the work is under review.
