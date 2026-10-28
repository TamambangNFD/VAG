# Vision Aid Glasses — Model Registry

Tracks every AI model used in the product: name, version, size, license,
runtime, accuracy/latency, and rollback plan. Updated every phase that
integrates or changes a model (Phases 3, 7, 10, 23, 26 per the roadmap).

---

## Object Detection — SSD MobileNetV2 FPN-Lite 320×320 (Phase 3)

| Field | Value |
|---|---|
| Model name | SSD MobileNetV2 FPN-Lite 320x320 (COCO-pretrained, quantized) |
| Source | TensorFlow Lite Object Detection model zoo / TensorFlow Model Garden |
| Version | TFLite metadata-packaged release, INT8 quantized |
| Runtime | LiteRT (TensorFlow Lite) via `org.tensorflow:tensorflow-lite` 2.16.1 + `tensorflow-lite-support`/`tensorflow-lite-metadata` 0.4.4, using the TFLite Task Library's `ObjectDetector` API |
| License | **Apache License 2.0** |
| Approx. model size | ~4-7 MB (quantized) — exact size depends on which published checkpoint is downloaded; confirm against the actual file once obtained |
| Accuracy / latency | Not yet measured on-device — vendor-published COCO mAP figures exist but have not been independently verified against this project's target phone. **Do not treat vendor benchmarks as this product's confirmed accuracy** until Phase 3 field testing (and later Phase 29) produces real numbers. |
| Rollback plan | None needed yet — this is the first model in this slot. If a future update regresses accuracy, revert to this same model/version by re-downloading it at this documented source and reverting `LiteRtObjectDetectionTool`'s `MODEL_ASSET_PATH` file to this version. |

### Why this model and license, not YOLO

YOLOv8 (Ultralytics) was considered and rejected for this slot because it is
distributed under **AGPL-3.0**, which carries commercial-use and
source-disclosure obligations unless a separate commercial license is
purchased from Ultralytics. Since Vision Aid Glasses' commercial-use
implications are not yet decided (see PRODUCT_SPEC.md non-goals/roadmap
Phase 34+), an Apache-2.0-licensed model avoids that risk entirely with no
downside for an initial SSD-class detector. This decision should be
revisited if a future phase has a specific accuracy requirement only YOLO
can meet — at that point, budget for an Ultralytics commercial license or
find another permissively licensed YOLO-family checkpoint.

### Known class-coverage gap — READ BEFORE RELYING ON THIS MODEL

COCO's category set does **not** include several object classes the
roadmap's Phase 0 target list asks for. This model can **never** produce:

- `DOOR`
- `STAIRS`
- `POLE`
- `ROAD`
- `SIDEWALK`
- `WALL`

This is enforced in code, not just documented: see
`CocoLabelMap.UNSUPPORTED_ROADMAP_LABELS` and its accompanying test in the
`vision` module, which fails if any COCO index is ever accidentally mapped
to one of these labels. Closing this gap requires one of:

1. A custom-trained/fine-tuned detection model (needs a licensed, provenance-
   documented dataset — see `datasets/DATASET_PROVENANCE.md` requirement).
2. A different technique entirely for some classes — e.g. semantic
   segmentation for ROAD/SIDEWALK, a dedicated stair-detection model.
3. Accepting the gap for V1 and revisiting at Phase 27 (Hardware Gap
   Analysis) or a dedicated future phase.

This gap does not block Phase 3 (which only requires *a* working detector),
but it must be carried forward honestly into every later phase that assumes
door/stair/pole/road/sidewalk/wall detection exists (notably Phase 16
Obstacle Avoidance and Phase 17 Traffic Safety) — those phases must either
work around this gap explicitly or trigger closing it first.

### How to obtain the model file

The actual `.tflite` binary is **not included in this repository** (it's a
multi-megabyte binary asset, and per the roadmap, model file provisioning is
the product owner's task). To use this implementation:

1. Obtain an SSD MobileNetV2 FPN-Lite 320x320 TFLite model with metadata
   (the TFLite Task Library requires the metadata-packaged variant, not a
   bare graph) from the official TensorFlow Lite / TensorFlow Model Garden
   distribution channels.
2. Verify its license file states Apache 2.0 (it should, as part of the
   TensorFlow project) before shipping it in the app.
3. Place it at: `android/VisionAid/vision/src/main/assets/ssd_mobilenet_v2_fpnlite_320.tflite`
   (this exact filename is referenced by
   `LiteRtObjectDetectionTool.MODEL_ASSET_PATH`).
4. Rebuild. `LiteRtObjectDetectionTool.availability()` will report
   `UNAVAILABLE_MODEL_NOT_LOADED` until the file is present and loads
   successfully — this was verified in the code path (see
   `HANDOFF.md`/`VERIFICATION.md`) but the actual load has not been tested
   against a real file in this sandbox.

---

## Face Detection — Short-Range Face Detector TFLite Model (Phase 7)

| Field | Value |
|---|---|
| Model name | Face detector (short-range), metadata-packaged TFLite, single-class ("face") SSD-style detector |
| Source | MediaPipe-family / TensorFlow Lite model distributions |
| Runtime | LiteRT via `org.tensorflow:tensorflow-lite-task-vision`'s `ObjectDetector` API — same loading pattern as Phase 3's object detector |
| License | **Apache License 2.0** |
| Accuracy / latency | Not yet measured on-device — same caveat as Phase 3's model. |
| Rollback plan | First model in this slot; none needed yet. |

This model only locates faces (bounding boxes) — it carries no identity
information and is not used for recognition. Its provenance concern is
therefore much lower than an identity-embedding model (see below): a
detector trained to answer "is there a face here" does not require the same
individual-consent scrutiny as a model trained to answer "whose face is
this," though the training images themselves still originate from people
who did not individually consent to this specific product's use — flagged
here for completeness, not glossed over, but treated as a materially lower
risk than the identification case below.

## Face Recognition / Identity Embedding — **NOT SELECTED. Deliberate gap.**

**No model is integrated for this. This is not an oversight or a TODO left
for later — it is a compliance decision, recorded here formally.**

The roadmap requires: *"any pretrained model used must have documented
training-data provenance and a license that permits this use. Do not use
any model or dataset without clear licensing."*

Commonly available pretrained face-embedding models were evaluated and
rejected on exactly this basis:

| Candidate | Problem |
|---|---|
| FaceNet / MobileFaceNet / ArcFace trained on **MS-Celeb-1M** | Dataset was **retracted by Microsoft in 2019** specifically because the people in it never consented to inclusion — a textbook biometric-privacy violation. Using a model derived from it would put this product in exactly the position the roadmap prohibits. |
| Models trained on **VGGFace2** or **CASIA-WebFace** | Built by scraping images of public figures without individual consent. Widely used in academic research, but not clearly licensed for shipping in a commercial product that identifies real people. |

Rather than pick one of these anyway to "make Phase 7 work," the
architecture was built with the identity step as an explicit, swappable gap
(`FaceEmbeddingModel` interface in `vision/face/`, currently only
implemented by `NoFaceEmbeddingModelAvailable`, which always reports
unavailable). `RealFaceRecognitionTool.availability()` correctly reports
`UNAVAILABLE_MODEL_NOT_LOADED` until this is resolved — it never fabricates
a recognition result.

**Paths to close this gap, in order of preference:**
1. Find a model with fully documented, consent-clean training data and a
   license that explicitly permits this use (keep looking — the field
   moves; a compliant option may exist that wasn't identified at the time
   of writing).
2. Train a custom, small embedding model on a dataset collected
   specifically for this product with proper informed consent — document
   per `datasets/DATASET_PROVENANCE.md`.
3. License a commercial on-device face-recognition SDK whose vendor
   agreement explicitly covers this product's use case.

Do not close this gap by adding a model without updating this section and
without the product owner's explicit review — biometric identification
carries real legal and ethical weight beyond ordinary technical debt.

---

## OCR Engine — Tesseract (Phase 8)

| Field | Value |
|---|---|
| Engine | Tesseract OCR, via the Tesseract4Android Android wrapper (`cz.adaptech.tesseract4android`) |
| Version | 4.7.0 (wrapper), Tesseract 5.x LSTM engine underneath |
| License | **Apache License 2.0** (both the wrapper and Tesseract itself) |
| Language data | `eng.traineddata` (English), `fra.traineddata` (French) — required by Section 0.5. Official Tesseract `tessdata`/`tessdata_best` repositories, also **Apache License 2.0**. |
| Runtime | Fully offline — no Google Play Services dependency of any kind |

### Why Tesseract over Google ML Kit Text Recognition

ML Kit's on-device text recognition is free and accurate, but depends on
Google Play Services being present on the device. Not every Android device
in the target market (Cameroon) is guaranteed to be Google Mobile Services
(GMS) certified — some budget devices ship without it. Since offline-first,
broad device compatibility is a stated product requirement
(`PRODUCT_SPEC.md`), Tesseract's zero-Play-Services dependency was judged
more important than ML Kit's likely-higher raw accuracy. This can be
revisited if field testing (Phase 29) shows Tesseract's accuracy is
insufficient and the target phone is confirmed to have reliable Play
Services access.

### How to obtain the language data files

The `.traineddata` files are **not included in this repository** (each is
several megabytes). To use this implementation:

1. Download `eng.traineddata` and `fra.traineddata` from the official
   Tesseract `tessdata` (faster, slightly less accurate) or `tessdata_best`
   (slower, more accurate) GitHub repository.
2. Place them at:
   `android/VisionAid/vision/src/main/assets/tessdata/eng.traineddata`
   `android/VisionAid/vision/src/main/assets/tessdata/fra.traineddata`
3. `TesseractOcrEngine.ensureLanguageInstalled()` copies them from assets to
   app-private storage on first use per language — this is a Tesseract
   requirement (it cannot read `.traineddata` directly from APK assets), not
   an extra step this project invented.
4. Until these files are present, OCR/document-reading correctly reports
   `UNAVAILABLE_MODEL_NOT_LOADED` / `ToolError.ModelNotLoaded` rather than
   failing silently or crashing.

### Known limitation carried into this phase: document edge detection

`SimpleDocumentDetector` (`vision/document/`) does **not** perform real
document-boundary detection — it assumes the document already fills the
frame. Genuine edge/corner detection (finding a page's boundary against a
cluttered background) typically requires a full CV library (e.g. OpenCV),
which was judged out of proportion for this phase. Perspective correction
(`PerspectiveCorrector`) and image enhancement (`ImageEnhancer`) are both
real, working implementations using Android's native `Matrix.setPolyToPoly`
and pixel-level contrast stretching respectively — only the corner-finding
step is a placeholder. This is documented, not hidden, and is a reasonable
target for a future phase if real-world testing shows users struggle to
frame documents correctly without automatic guidance.
