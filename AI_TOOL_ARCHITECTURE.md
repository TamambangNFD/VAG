# Vision Aid Glasses — AI Tool Architecture (AI_TOOL_ARCHITECTURE.md)

**Phase:** 2 — AI Tool Architecture
**Module:** `android/VisionAid/ai/`

## Purpose

This phase defines a stable, swappable interface for every AI capability the
product needs, so a real model (Phase 3 onward) can be dropped in — or later
replaced — without touching any calling code. No real model is integrated in
this phase; every tool below ships only with a deterministic mock.

## Shared Contracts (`ai/common/`)

Every tool interface is built from the same small set of shared types, so
callers (the future Agent, Situation Engine, and WorldModel) can reason about
any tool the same way:

| Type | Purpose |
|---|---|
| `ToolResult<T>` | Universal envelope: `Success(data, confidence, timestampMillis, metadata)` or `Failure(error, timestampMillis)`. |
| `Confidence` / `ConfidenceLevel` | `UNKNOWN / LOW / MEDIUM / HIGH`, plus an optional raw `score`. `UNKNOWN` is a first-class, expected outcome — not an error. |
| `ToolError` | Sealed class for genuine failures (`ModelNotLoaded`, `HardwareUnavailable`, `Timeout`, `InvalidInput`, `InternalError`). Never used for "nothing found" or "low confidence" — those are successful results. |
| `ToolAvailability` | Whether the tool can run at all right now (`AVAILABLE`, `UNAVAILABLE_HARDWARE_MISSING`, `UNAVAILABLE_MODEL_NOT_LOADED`, `UNAVAILABLE_PERMISSION_DENIED`, `DISABLED_BY_USER`, `ERROR`). Distinct from a per-request `ToolError`. |
| `OperationMode` | `OFFLINE_ONLY / OFFLINE_WITH_ONLINE_ENHANCEMENT / ONLINE_ONLY`. Every tool declares this so the future Network Capability Manager (Phase 22) and Agent (Phase 13) know what still works offline. |
| `FrameContext` | Shared input envelope for frame-based tools: `frameId`, opaque `imageData`, `captureTimestampMillis`, and an explicit `LocaleTag` (never hard-coded, per Section 0.5). |

## Tool Interfaces

| Tool | Input | Output | Offline mode | Notes |
|---|---|---|---|---|
| `ObjectDetectionTool` | `FrameContext` | `DetectionOutput` (list of `DetectedObject` with label/box/confidence) | OFFLINE_ONLY | Phase 3 replaces mock with a real on-device model. |
| `ObjectTrackingTool` | detections + timestamp | `TrackingOutput` (stable `trackId`s, movement direction) | OFFLINE_ONLY | Owns its own track-history state; Phase 4 replaces with ByteTrack or similar. |
| `DepthTool` | `FrameContext` + object IDs | `DepthOutput` (near/medium/far/**UNKNOWN** per object) | OFFLINE_ONLY | **Safety-critical.** Mock always returns UNKNOWN by design — see below. |
| `FaceRecognitionTool` | `FrameContext` (+ name for enroll) | `RecognitionOutput` / `EnrollmentOutput` | OFFLINE_ONLY | Unknown faces reported as `personId = null`, never guessed. Consent capture happens in the caller before `enroll()`. |
| `OCRTool` | `FrameContext` | `OcrOutput` (text blocks, per-block confidence) | OFFLINE_ONLY | Locale-aware (French/English) via `FrameContext.locale`. |
| `CurrencyRecognitionTool` | `FrameContext` | `CurrencyOutput` (denomination or null, `imageQualitySufficient`) | OFFLINE_ONLY | XAF only for now; designed for more currencies later without interface changes. |
| `NavigationTool` | lat/lng + destination query | `RouteOutput` (turn-by-turn steps) | OFFLINE_WITH_ONLINE_ENHANCEMENT | Only navigation tool not driven by a camera frame. |
| `DocumentTool` | `FrameContext` (load) / no input (navigate) | `DocumentOutput` / `Paragraph` | OFFLINE_ONLY | Stateful: `loadDocument` → `readNext`/`readPrevious`/`readAll`. |
| `SafetyTool` | tracked objects + distance bands | `SafetyOutput` (`SAFE_CANDIDATE/CAUTION/DANGER/STOP/UNKNOWN` + reason) | OFFLINE_ONLY | **Safety-critical.** Mock always returns `UNKNOWN` — see below. |
| `SpeechRecognitionTool` | raw audio bytes + `LocaleTag` | `SpeechRecognitionOutput` (transcript) | OFFLINE_ONLY | Phase 11 replaces mock with a real ASR engine (e.g. whisper.cpp). |
| `TextToSpeechTool` | text + locale + `SpeechPriority` | `ToolResult<Unit>` | OFFLINE_ONLY | Priority ordering `NORMAL < IMPORTANT < WARNING < CRITICAL < EMERGENCY`; mock records a spoken-history queue for testing. |

## Safety-Critical Design Choice (Section 0.4 of the roadmap)

`DepthTool` and `SafetyTool` are the two safety-critical interfaces defined
in this phase (full implementations arrive at Phases 5 and 16/17
respectively). Their **mock implementations deliberately always return
`UNKNOWN`** rather than any plausible-looking value. This is not an
oversight — a mock with no real model behind it that returned, say,
`DistanceBand.MEDIUM` or `SafetyState.SAFE_CANDIDATE` would be exactly the
"confident-but-wrong" failure mode the roadmap prohibits. Every downstream
consumer built against these mocks during Phases 3–15 will therefore be
forced to handle the `UNKNOWN` case correctly from day one, rather than
discovering a gap when the real safety pipeline lands.

## Why Interfaces Live in the `ai` Module

Per `ARCHITECTURE.md`, the `ai` module was reserved in Phase 1 specifically
to hold these tool contracts. Concrete implementations will move into their
natural home module as each phase builds them (e.g. `ObjectDetectionTool`'s
real implementation lands in `vision` at Phase 3, `CurrencyRecognitionTool`'s
in `currency` at Phase 10) — those modules already depend on `ai` (or will be
updated to) so they can implement its interfaces without `ai` ever depending
back on them.

## What This Phase Does NOT Include

- No real models — every implementation is a deterministic mock.
- No dependency injection wiring into `MainActivity` — that's an application-
  layer decision for a later phase once real tools exist to inject.
- No persistence — `FaceRecognitionTool`'s mock enrollment is in-memory only;
  real encrypted storage arrives in Phase 7/21.
