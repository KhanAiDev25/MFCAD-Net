# MFCAD-Net

**Turning hand-drawn mechanical sketches into parametric CAD geometry.**

Photograph a sketch on paper. Get back typed geometric primitives — lines, circles and arcs with real parameters — instead of a picture that merely looks like a drawing.

## The problem

A generative model can produce something that looks like a clean drawing. That is not the same as producing geometry you can machine from. A CAD file needs each entity to be a typed primitive with parameters: this is a circle at (x, y) of radius r, not a plausible arrangement of dark pixels.

The requirement is exactness, and it rules out most of the obvious approaches.

## What the system does

- Takes an uncalibrated photograph of a hand-drawn mechanical sketch as input.
- Recovers lines, circles and arcs as parametric entities rather than as raster output.
- Applies CAD-style regularisation so the result obeys the conventions real drawings follow rather than reproducing the unsteadiness of a freehand stroke.
- Places dimension labels on the reconstructed geometry automatically.
- Ships with an interactive demonstrator for reviewing results.

Evaluated on prismatic mechanical parts. Organic and freeform geometry is out of scope.

## Status

Active MS research. The method, quantitative evaluation and ablation studies are being prepared for publication and are not included here. Training code, datasets and model weights are not published.

## Context

MS Artificial Intelligence thesis, University of Engineering and Technology (UET) Peshawar, Department of Electrical Engineering.

The author spent six years programming 8-axis Siemens Sinumerik CNC machines and teaching CNC and CAM before moving into AI, which is the reason this project treats machinable output rather than plausible output as the bar.

---

Questions, collaboration and research enquiries are welcome via the contact details on my GitHub profile.
