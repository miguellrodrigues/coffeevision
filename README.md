# CoffeeVision

![CoffeeVision — any model can paint a field green; this one won't colour a cell it can't prove](assets/01-coffeevision-hero.webp)

**A coffee-maturity map that refuses to colour a cell it cannot prove.**

CoffeeVision detects individual cherries in field photographs, classifies their
maturity stage, and aggregates the result into an H3 map. The interesting part is
not the detector — it is everything that stops the map from lying.

> This repository is the public, media-only presentation of the project. It holds
> the case-study images and technical diagrams, not the application, training data,
> model weights, or private field coordinates.

## The problem with green maps

A model that scores every photograph and paints the surrounding hectare will
produce a beautiful, confident, and largely fictional map. Sampling density gets
read as crop condition. Cells nobody visited get interpolated from cells that were.
A single blurry frame with bad GPS moves an entire block.

CoffeeVision treats that as the actual engineering problem:

| Rule | Consequence on the map |
| --- | --- |
| Minimum photographic evidence per cell | Under-sampled cells stay hatched, never coloured |
| GPS accuracy gate | Observations with incompatible precision are rejected outright |
| No interpolation | Unsampled area stays blank — absence is not zero |
| Repeat-frame detection | Near-duplicate photographs of one branch count once |
| Every cell stays auditable | Any colour opens the photographs that produced it |

## From field photograph to spatial evidence

<p align="center">
  <img src="assets/02-desafio-no-campo.webp" width="49%" alt="Coffee branch under natural field conditions">
  <img src="assets/03-deteccao-e-maturidade.webp" width="49%" alt="Detected coffee fruit with maturity classes">
</p>

Real scenes bring occlusion, mixed maturity on one branch, small fruit, hard shadow,
and dense clusters. The original photograph stays attached to every prediction, so a
summary can always be traced back to what produced it.

<p align="center">
  <img src="assets/04-mapa-de-maturacao.webp" width="49%" alt="CoffeeVision maturity map">
  <img src="assets/05-evidencia-da-celula.webp" width="49%" alt="Auditable photo evidence for a selected H3 cell">
</p>

Selecting a cell does not just report a number. It reports a number, an interval,
and the frames behind it.

![Fruit distribution analysis for a selected cell](assets/06-distribuicoes-dos-frutos.webp)

## What a single cell reports

Taking one r11 cell from the demonstration set:

| Reading | Value |
| --- | --- |
| Green | 78% — 154 fruit, CI 72–83% |
| Turning | 12% — 24 fruit, CI 8–17% |
| Ripe | 10% — 19 fruit, CI 6–15% |
| Sampling coverage | 9 independent photographs of 10 — 1 repeat excluded |
| Mean detection confidence | 85% |
| Mean maturity confidence | 91% |

Class shares carry confidence intervals because nine photographs of one 57 m cell
are a sample, not a census. Apparent fruit size is reported as a share of the frame
and labelled as such — it is not a physical diameter, and perspective, distance, and
camera resolution all move it.

## Field-level evidence accounting

| | |
| --- | ---: |
| Photographic observations | 216 |
| Fruit detections | 5,849 |
| Cells coloured | 14 |
| Cells withheld for low evidence | 2 |
| Observations rejected on GPS accuracy | 39 |
| Median GPS accuracy | 5.5 m |
| Map resolution | H3 r11 · approximately 57 m |

These describe the demonstration dataset — a deterministic 50% sample of the local
annotated data plus four project-root test images. **They are not model-validation
metrics; a held-out evaluation is not yet published.**

## How the geometry stays honest

![GPS observation to an evidence-aware H3 field map](assets/07-geospatial-method.svg)

The recorded coordinate is where the camera stood, not where the fruit grew. At r11
that distinction is inside one cell, which is precisely why the resolution was chosen
and why finer grids are refused rather than rendered.

## Architecture

![CoffeeVision system architecture](assets/08-system-architecture.svg)

Capture, quality checks, inference, persistence, spatial aggregation, and
presentation stay separated. Model version and prediction latency are stored
alongside every observation, so a map result traces back to both its evidence and
the inference configuration that produced it.

`Python` · `PyTorch` · `Ultralytics` · `OpenCV` · `FastAPI` · `PostgreSQL` ·
`PostGIS` · `H3` · `React` · `TypeScript` · `MapLibre` · `Docker`

## What it does not do

CoffeeVision supports visual sampling. It does not infer whole-farm yield from one
image, diagnose disease without suitable labels, or automate harvest decisions
without human review. Production use additionally requires a reviewed field bridge
set, approved dataset and model licensing, and explicit authorisation for any precise
farm coordinates.

## Provenance

Map coordinates in every published screen are deliberately synthetic and displaced;
the warning stays visible in the interface rather than only in this file. The field
photograph in the compositions was supplied by the project owner and is cleared
without restriction for portfolio use. Product screens are direct captures of the
locally running application, not mockups.

---

Designed and developed by [Miguel L. Rodrigues](https://miguellrodrigues.github.io/).
