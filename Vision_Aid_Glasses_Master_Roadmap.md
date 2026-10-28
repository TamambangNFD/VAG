# Vision Aid Glasses — Master Product & Development Roadmap

*Updated version — includes continuity system (Handoff / Progress / Verification), git checkpoint convention, broadened safety-critical scope, dataset provenance requirements, coding standards, secrets management, and localization requirements.*

## Purpose

Vision Aid Glasses is an offline-first wearable AI perception and assistance system for blind and low-vision users.

Core concept:

> The glasses provide the user's eyes, ears, sensors and interface. The phone provides most of the AI computation, storage, battery and connectivity. The system should work offline for essential functions and use the Internet only when it improves capability.

The final system should not try to describe everything. It should understand what matters to the user at that moment and communicate concise, useful information through audio and, where supported, visual output.

---

# 0. Ground Rules for Building With AI

## How to use an AI coding agent

Build **one phase at a time**.

Never give an AI agent the instruction:

> "Build the entire Vision Aid Glasses project."

Instead, for every phase:

1. Give it the phase specification (from this document).
2. Let it inspect the existing project.
3. Confirm the **Definition of Ready** (see Section 0.1) — the previous phase's outputs actually exist and work.
4. Ask it to implement only that phase.
5. Require tests.
6. Run the application.
7. Fix all errors.
8. Review the architecture.
9. Complete the **Standard Phase Closing Protocol** (Section 0.1): update `HANDOFF.md`, `PROGRESS.md`, `VERIFICATION.md`.
10. Git checkpoint (commit + tag, Section 0.2).
11. Only then move to the next phase.

### Rules for the AI

The coding AI must:

- never rewrite unrelated working code;
- never invent APIs or SDK functions;
- inspect official documentation when integrating hardware;
- keep modules loosely coupled;
- document interfaces;
- write tests;
- avoid hard-coding secrets — use the config/secrets pattern established in Phase 1 (Section 0.3);
- follow `CODING_STANDARDS.md` (created in Phase 1) for naming, style, and error handling;
- preserve offline functionality;
- log errors safely;
- avoid collecting unnecessary personal data;
- clearly mark experimental/safety-critical functionality (see broadened safety-critical scope, Section 0.4);
- never claim that an unsafe feature is production-ready without testing;
- ask for clarification when a hardware/API requirement is genuinely unknown;
- always complete the Standard Phase Opening and Closing Protocols in Section 0.1.

---

## 0.1 Project Continuity System (Handoff / Progress / Verification)

This is the system that lets you say **"build next phase"** or **"continue with the next"** in any chat — new or existing — and have the AI pick up exactly where the last session left off, without you re-explaining the project.

It relies on three living files at the repo root, plus a per-phase verification record.

### `PROGRESS.md`

A single running table, updated at the end of every phase. This is the fastest way for a new session to see project status at a glance.

```markdown
# Vision Aid Glasses — Progress Tracker

| Phase | Name                          | Status        | Date       | Notes                          |
|-------|-------------------------------|---------------|------------|---------------------------------|
| 0     | Product Definition            | Complete      | 2026-01-10 | PRODUCT_SPEC.md approved        |
| 1     | Android Foundation             | Complete      | 2026-01-14 | Builds/launches on Pixel test   |
| 2     | AI Tool Architecture           | In Progress   | 2026-01-15 | Mocks done, tests pending       |
| 3     | Object Detection               | Not Started   | —          | Blocked on model license choice |
| ...   | ...                            | ...           | ...        | ...                              |

Status values: Not Started | In Progress | Blocked | Complete | Skipped
```

### `HANDOFF.md`

Overwritten (or appended, your choice — pick one convention and stick to it) at the end of **every** phase. This is the single most important file for resuming work in a new chat. It should always answer: *what exists right now, what was decided, what's next, and what does the next session need to know before touching anything.*

```markdown
# Vision Aid Glasses — Latest Handoff

## Last completed phase
Phase N — <Phase Name>

## What was built
- <summary of what was implemented>
- <key files/modules touched>

## Decisions made this phase
- <decision 1 and why>
- <decision 2 and why>

## Interfaces exposed (for next phases to depend on)
- <ToolName>: <input contract> -> <output contract>

## Known issues / limitations
- <issue 1>
- <issue 2>

## What the next phase needs to know
- <dependency or constraint the next agent must respect>

## Git checkpoint
Tag: phase-N-complete
Commit: <hash>

## Verification status
See VERIFICATION.md — Phase N: PASSED / PASSED WITH NOTES / FAILED
```

### `VERIFICATION.md`

One file with one section per phase (or a `docs/verification/phase-N.md` file per phase if it gets long — pick one convention early and keep it consistent). This is where acceptance criteria get checked off, not just written.

```markdown
# Phase N — <Phase Name> — Verification

## Acceptance criteria (from phase spec)
- [ ] Criterion 1
- [ ] Criterion 2

## Tests run
- Unit tests: <pass/fail count>
- Manual test: <what was manually verified and how>

## Result
PASSED / PASSED WITH NOTES / FAILED

## Notes / deviations
<anything that didn't go exactly as specified, and why>

## Signed off by
<you — date>
```

### Standard Phase Opening Protocol (Definition of Ready)

Before starting any phase, the AI must:

1. Read `PROGRESS.md` and `HANDOFF.md`.
2. Confirm the previous phase's status is `Complete` in `PROGRESS.md`. If not, stop and flag it.
3. Confirm the interfaces the new phase depends on (listed in the previous `HANDOFF.md`) actually exist in the codebase.
4. State its understanding of current project state back to you in 3-5 sentences before writing code.

### Standard Phase Closing Protocol

At the end of any phase, the AI must:

1. Update `PROGRESS.md` (mark phase Complete/Blocked, add date and one-line note).
2. Overwrite `HANDOFF.md` with the new phase's summary (using the template above).
3. Add or update the phase's section in `VERIFICATION.md`, checking off acceptance criteria against actual test results — not just restating the spec.
4. Git commit and tag (Section 0.2).
5. Report to you: what changed, what passed verification, what didn't, and what phase is next.

### How "build next phase" works

Once this system is in place, you only need to say **"build next phase"** or **"continue with the next"**. The AI's first move is always: read `PROGRESS.md` + `HANDOFF.md` → run the Opening Protocol → identify the next incomplete phase → confirm with you if anything is ambiguous → proceed.

### Resuming in a new chat (same project)

If you start a **new chat inside the same project**, you don't need to re-upload the whole codebase or re-explain history. Upload only the **latest builds pack**:

1. `PROGRESS.md`
2. `HANDOFF.md` (latest)
3. `VERIFICATION.md` (or the latest phase's verification file)
4. `CODING_STANDARDS.md`
5. `ARCHITECTURE.md` (if it changed since last upload)
6. Optionally: `MODEL_REGISTRY.md` and `HARDWARE_GAP_ANALYSIS.md` if the new phase touches models or hardware

Then say: *"Here's the latest state of Vision Aid Glasses. Continue with the next phase."* The AI reads these five-to-six files and has everything it needs — you do not need to re-upload the full source tree unless the AI asks to inspect specific files.

---

## 0.2 Git Checkpoint Convention

- One commit per completed phase, minimum.
- Commit message format: `Phase N: <short description>` (e.g. `Phase 3: offline object detection with LiteRT`).
- Tag format: `phase-N-complete` (e.g. `phase-3-complete`).
- Branch convention: work directly on `main` for early phases if you're solo; once past Phase 13 (Agent) consider `phase/N-name` branches merged into `main` only after verification passes.
- Never tag a phase `-complete` until `VERIFICATION.md` shows PASSED or PASSED WITH NOTES for that phase.
- Rolling back: `git checkout phase-(N-1)-complete` always gives you a known-good state.

---

## 0.3 Secrets & Configuration Management (established in Phase 1)

- No API keys, tokens, or credentials committed to the repository, ever — including in tests or sample config.
- Local secrets go in a git-ignored `local.properties` / `.env` file with a checked-in `.env.example` showing required keys with placeholder values.
- Any future online service (Phase 22 onward) reads credentials from this config layer, never hard-coded.
- Document the config pattern in `ARCHITECTURE.md` once established in Phase 1.

---

## 0.4 Safety-Critical Scope (broadened)

The following are treated as **safety-critical** and must always prefer `UNKNOWN`/conservative output over a confident-but-wrong result:

- Phase 5 — Depth and Distance
- Phase 16 — Obstacle Avoidance
- Phase 17 — Traffic Safety Engine
- Phase 18 — Emergency Interruption

A general-purpose LLM/VLM must never be the sole decision-maker for any of these four phases. Each must expose a deterministic, testable output with a documented confidence/uncertainty state.

---

## 0.5 Localization

Cameroon (Douala) is a primary target market. From Phase 0 onward:

- ASR and TTS must support **French and English** at minimum (Cameroon's official languages), with the architecture left open for additional local languages later.
- Currency recognition (Phase 10) targets CFA franc (XAF) first, per Phase 10 spec.
- Do not hard-code language assumptions into any tool interface — pass language/locale as a parameter.

---

# 1. PHASE 0 — Product Definition

## Goal

Freeze the product requirements before coding.

Define:

### Low Vision Mode

- magnification
- text enlargement
- OCR
- face identification
- object recognition
- colors
- document reading

### Blind Assistance Mode

- obstacle detection
- object identification
- people identification
- currency recognition
- document reading
- environmental description
- navigation

### Safety Mode

- obstacle warnings
- moving vehicle detection
- pedestrian detection
- stairs/drop-off detection
- road awareness
- emergency interruption

### General requirements

- offline-first
- Android phone as main compute device
- smart glasses as wearable interface
- Bluetooth/Wi-Fi communication
- optional Internet
- voice-first interaction
- local encrypted memory
- privacy controls
- low latency
- low power
- modular AI architecture
- **localization: French and English support at minimum (see Section 0.5)**

## Who does what?

**Me:** Decide product requirements, target users, priorities, acceptable behavior and constraints.

**AI:** Convert the decisions into a formal technical specification.

## Prompt to AI

```text
You are the lead systems architect for a project called Vision Aid Glasses.

We are building an offline-first assistive wearable system for blind and low-vision users.

Architecture:
- Smart glasses provide cameras, microphones, speakers/earphones, display where available, IMU and other supported sensors.
- An Android phone performs most AI inference, application logic, storage, GPS and connectivity.
- Internet is optional and must enhance rather than replace core offline functionality.
- A Jetson or other external compute device is optional for development and heavy experiments, but do not make it mandatory unless justified.
- The first product should use existing smart-glasses hardware such as Vuzix or Rokid rather than immediately designing custom glasses.
- ASR/TTS must support French and English at minimum, with locale passed as a parameter rather than hard-coded.

Create a formal PRODUCT_SPEC.md covering:
1. Product vision
2. Target users
3. Low Vision Mode
4. Blind Assistance Mode
5. Safety Mode
6. Offline requirements
7. Online capabilities
8. Privacy requirements
9. Hardware assumptions
10. Software requirements
11. Non-goals
12. Safety limitations
13. MVP/V1/V2/V3 feature boundaries
14. Acceptance criteria
15. Localization requirements

Do not write application code yet.
Do not invent hardware capabilities.
Where a capability depends on the selected glasses model, mark it as TBD.

Before finishing, follow the Standard Phase Closing Protocol: create PROGRESS.md and HANDOFF.md at repo root reflecting Phase 0 completion, and record acceptance criteria in VERIFICATION.md.
```

---

# 2. PHASE 1 — Android Software Foundation

## Goal

Create the phone application foundation without AI models.

Recommended initial stack:

- Android
- Kotlin
- Jetpack architecture
- Camera APIs
- Bluetooth
- Wi-Fi
- GPS/location
- microphone
- local storage/database
- logging
- testing framework

Suggested structure:

```text
VisionAid/
├── app/
├── core/
├── ai/
├── vision/
├── audio/
├── navigation/
├── safety/
├── memory/
├── documents/
├── currency/
├── glasses/
└── tests/
```

## Who does what?

**Me:**
- create the project/repository;
- select the initial Android device;
- provide hardware/API documentation;
- run builds and physically test the application.

**AI:**
- create the Android architecture;
- create modules;
- configure permissions;
- create navigation/settings;
- implement logging;
- create tests;
- establish the secrets/config pattern (Section 0.3);
- create `CODING_STANDARDS.md`.

## Prompt

```text
Build Phase 1 of Vision Aid Glasses.

First, confirm the Definition of Ready: read PROGRESS.md and HANDOFF.md and confirm Phase 0 is marked Complete with PRODUCT_SPEC.md approved.

Create a clean Android/Kotlin foundation for the Vision Aid application.

Requirements:
- modular architecture;
- separate core, AI, vision, audio, navigation, safety, memory, documents, currency and glasses modules where practical;
- camera permission;
- microphone permission;
- Bluetooth permission;
- location permission;
- local storage;
- application settings;
- structured logging;
- test infrastructure;
- clear dependency boundaries;
- a git-ignored local config/secrets pattern (.env or local.properties) with a checked-in .env.example — no secrets ever committed.

Do NOT implement AI models yet.
Do NOT integrate glasses SDKs yet.
Do NOT add cloud AI.

The application must compile and launch successfully.

Create:
- architecture documentation (ARCHITECTURE.md);
- module documentation;
- basic unit tests;
- a README explaining how to build and run it;
- CODING_STANDARDS.md covering naming, style, and error-handling conventions for all future phases.

Before modifying anything, inspect the existing repository and preserve working code.

Before finishing, follow the Standard Phase Closing Protocol: update PROGRESS.md, HANDOFF.md, and VERIFICATION.md, then git commit and tag as phase-1-complete.
```

---

# 3. PHASE 2 — AI Tool Architecture

## Goal

Create a standard interface for AI capabilities.

The agent should eventually call tools such as:

```text
ObjectDetectionTool
TrackingTool
DepthTool
FaceRecognitionTool
OCRTool
CurrencyTool
NavigationTool
DocumentTool
SafetyTool
```

Example interface:

```text
Input:
image/frame/context

Output:
structured result
confidence
timestamp
metadata
```

## Why?

You should be able to replace YOLO later without rebuilding the entire application.

## Who does what?

**Me:** approve interfaces and behavior.

**AI:** implement interfaces, data classes, dependency injection and tests.

## Prompt

```text
Build Phase 2: Vision Aid AI Tool Architecture.

Confirm Definition of Ready: Phase 1 marked Complete in PROGRESS.md, project compiles and launches per Phase 1 HANDOFF.md.

Create clean interfaces for:
- ObjectDetectionTool
- ObjectTrackingTool
- DepthTool
- FaceRecognitionTool
- OCRTool
- CurrencyRecognitionTool
- NavigationTool
- DocumentTool
- SafetyTool
- SpeechRecognitionTool
- TextToSpeechTool

For each tool define:
- input contract;
- output contract;
- confidence representation;
- timestamp;
- error states;
- availability state;
- offline/online capability.

Use interfaces so models can be replaced later.
Follow CODING_STANDARDS.md.

Do not integrate real AI models yet.

Create mock implementations so the application can demonstrate the complete data flow.

Write unit tests for each interface.

Document the architecture in AI_TOOL_ARCHITECTURE.md.

Follow the Standard Phase Closing Protocol before finishing (PROGRESS.md, HANDOFF.md, VERIFICATION.md, git tag phase-2-complete).
```

---

# 4. PHASE 3 — Object Detection

## Goal

Add the first real computer-vision model.

Investigate lightweight YOLO deployment for Android using a suitable mobile inference runtime such as LiteRT or ONNX Runtime.

Start with important objects:

- person
- car
- motorcycle
- bicycle
- chair
- table
- door
- stairs
- pole
- road
- sidewalk
- bag
- bottle
- wall
- vehicle

## Who does what?

**Me:**
- choose the model/license;
- download or provide model files;
- test camera performance on the actual phone.

**AI:**
- integrate the inference runtime;
- preprocess frames;
- run inference;
- return structured detections;
- optimize where possible;
- test latency.

## Prompt

```text
Build Phase 3: Offline Object Detection.

Confirm Definition of Ready: Phase 2 Complete, ObjectDetectionTool mock interface exists per Phase 2 HANDOFF.md.

Integrate a lightweight object-detection model suitable for Android offline inference.

Before selecting a model:
1. Check the model's license.
2. Confirm commercial-use implications.
3. Confirm Android deployment support.
4. Prefer a model that can run efficiently on-device.

Implement:
camera frame -> preprocessing -> inference -> postprocessing -> ObjectDetectionTool.

Return:
- class
- confidence
- bounding box
- timestamp
- frame ID

Do not implement spoken alerts yet.

Do not claim the model is production-ready.

Add:
- unit tests;
- inference tests;
- performance logging;
- model-loading error handling;
- documentation.
- an entry in MODEL_REGISTRY.md (create if it doesn't exist) recording model name, version, size, license, runtime, and accuracy/latency numbers.

The system must continue functioning offline.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-3-complete).
```

---

# 5. PHASE 4 — Object Tracking

## Goal

Track the same object across frames.

Example:

```text
Car #12
Distance: unknown
Direction: left -> right
Movement: approaching/away
```

Possible technology:

- ByteTrack or another suitable lightweight tracker.

## Prompt

```text
Build Phase 4: Multi-object Tracking.

Confirm Definition of Ready: Phase 3 Complete, ObjectDetectionTool returns real detections per Phase 3 HANDOFF.md.

Use the existing object-detection output and add a tracking layer.

Requirements:
- assign stable temporary object IDs;
- associate detections between frames;
- estimate movement direction;
- estimate object persistence;
- handle objects entering/leaving the frame;
- handle missed detections;
- avoid creating excessive duplicate IDs.

Do not add traffic safety decisions yet.

Expose a clean TrackingTool interface.

Create tests using recorded detection sequences.

Document limitations and expected failure cases.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-4-complete).
```

---

# 6. PHASE 5 — Depth and Distance

**This phase is safety-critical (Section 0.4). Prefer near/medium/far uncertainty over false precision.**

## Goal

Estimate how far important objects are from the user.

Primary approach:

- stereo cameras where available;
- stereo processing;
- optional monocular depth estimation as a secondary signal.

Concept:

```text
Left camera + right camera
        ↓
stereo matching
        ↓
depth
        ↓
object distance
```

## Important

Do not claim centimeter accuracy without calibration and validation.

## Prompt

```text
Build Phase 5: Depth and Distance.

Confirm Definition of Ready: Phase 4 Complete, TrackingTool produces stable IDs per Phase 4 HANDOFF.md.

This phase is safety-critical. Prefer near/medium/far classification and an explicit UNKNOWN/low-confidence state over any imprecise numeric claim.

Create a modular DepthTool.

Initial implementation:
- support stereo camera input when the selected glasses expose suitable synchronized cameras;
- implement camera calibration configuration;
- generate depth information;
- associate depth with detected objects;
- classify distance into near/medium/far before attempting precise measurements.

Optionally create a pluggable interface for a monocular depth model.

Do not assume the selected glasses provide two synchronized cameras.
Create a hardware capability abstraction.

Add tests using recorded stereo images.

Document calibration requirements, accuracy limitations and failure conditions.

Do not implement road-crossing decisions.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-5-complete). In VERIFICATION.md, explicitly record false-negative/false-positive behavior under uncertainty since this phase is safety-critical.
```

---

# 7. PHASE 6 — Spatial World Model

## Goal

Combine:

- object
- position
- distance
- movement
- confidence
- timestamp

into a local representation of the user's surroundings.

Example:

```text
Pole
distance: 1.1 m
position: center
movement: stationary
confidence: high
```

## Prompt

```text
Build Phase 6: Spatial World Model.

Confirm Definition of Ready: Phase 5 Complete, DepthTool classifies near/medium/far per Phase 5 HANDOFF.md.

Create a local real-time WorldModel that combines:
- object detections;
- tracking;
- depth;
- camera position;
- timestamps;
- confidence.

Represent important nearby objects using a structured data model.

Support queries such as:
- nearest obstacle;
- objects in walking path;
- objects left/right/center;
- moving objects;
- objects within configurable distance.

Do not generate natural-language responses yet.

Create tests using synthetic scenes.

Prioritize deterministic, explainable data structures over complex AI.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-6-complete).
```

---

# 8. PHASE 7 — Face Detection and Recognition

## Goal

Allow the user to register known people and recognize them locally.

Pipeline:

```text
camera
↓
face detection
↓
face embedding
↓
encrypted local database
↓
matching
```

## Prompt

```text
Build Phase 7: Local Face Recognition.

Confirm Definition of Ready: Phase 6 Complete, WorldModel queries work per Phase 6 HANDOFF.md.

Implement:
1. face detection;
2. face embedding generation;
3. local encrypted face-profile storage;
4. enrollment;
5. recognition;
6. confidence thresholding;
7. unknown-person handling.

Commands should include:
- enroll person;
- rename person;
- remove person;
- temporarily disable recognition.

Do not upload face data to a cloud service.

Do not identify unknown people from the Internet.

Return confidence and never claim identity when confidence is below the configured threshold.

Dataset/provenance requirement: any pretrained face-embedding model used must have documented training-data provenance and a license that permits this use. Do not use any model or dataset without clear licensing. Enrollment data collected from the user is local-only and requires explicit user consent captured in-app before storage.

Create privacy documentation and tests.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-7-complete).
```

---

# 9. PHASE 8 — OCR and Document Reading

## Goal

Read documents completely offline.

Pipeline:

```text
camera
↓
document detection
↓
perspective correction
↓
image enhancement
↓
OCR
↓
text
↓
document structure
```

## Prompt

```text
Build Phase 8: Offline OCR and Document Reading.

Confirm Definition of Ready: Phase 7 Complete per Phase 7 HANDOFF.md.

Evaluate suitable OCR engines for Android offline use.

Implement:
- document detection;
- crop;
- perspective correction;
- image enhancement;
- OCR;
- text ordering;
- paragraph extraction;
- page handling.

Support commands:
- Read this;
- Read next;
- Read previous;
- Read all;
- What does this say?

Support French and English OCR output at minimum (Section 0.5).

Do not add cloud OCR.

Create performance tests using sample documents.

The result must be exposed through DocumentTool and OCRTool.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-8-complete).
```

---

# 10. PHASE 9 — Low Vision Magnification

## Goal

Provide visual enlargement where the selected glasses support a display.

Features:

- 2x
- 4x
- 8x
- focus/region selection
- enhancement

## Prompt

```text
Build Phase 9: Low Vision Magnification.

Confirm Definition of Ready: Phase 8 Complete per Phase 8 HANDOFF.md.

Implement a magnification pipeline:
camera frame -> crop/region of interest -> enhancement -> scaling -> glasses display.

Support:
- zoom levels;
- digital crop;
- contrast enhancement;
- brightness adjustment;
- text-focused mode.

The implementation must be abstracted from the specific glasses SDK.

If the selected glasses do not expose a suitable display API, implement a phone-side test mode and clearly document the hardware limitation.

Do not claim that digital zoom can recover detail that the camera did not capture.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-9-complete).
```

---

# 11. PHASE 10 — Currency Recognition

## Goal

Start with FCFA and later expand.

## Prompt

```text
Build Phase 10: Currency Recognition.

Confirm Definition of Ready: Phase 9 Complete per Phase 9 HANDOFF.md.

Create a dedicated CurrencyRecognitionTool.

Initial target:
- Cameroon CFA franc (XAF) banknotes and coins where reliable training data is available.

Requirements:
- denomination classification;
- confidence;
- unknown/uncertain result;
- offline operation;
- image-quality validation.

Never state a low-confidence result as certain.

Dataset/provenance requirement: document the source, licensing, and collection method of any currency image dataset used for training or fine-tuning. Do not use scraped or unlicensed imagery without verifying usage rights.

Create the model interface so additional currencies can be added later.

Do not implement counterfeit detection in this phase.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-10-complete).
```

---

# 12. PHASE 11 — Offline Speech Recognition

## Goal

Turn the microphone into the user's primary command interface.

Potential technology to evaluate:

- whisper.cpp
- another efficient offline ASR engine.

## Prompt

```text
Build Phase 11: Offline Speech Recognition.

Confirm Definition of Ready: Phase 10 Complete per Phase 10 HANDOFF.md.

Evaluate an offline Android speech-recognition engine.

Requirements:
- microphone input;
- voice activity detection;
- speech-to-text;
- configurable language (French and English at minimum, per Section 0.5);
- low-latency operation;
- no Internet requirement;
- error handling.

Support natural commands such as:
- What is in front of me?
- Read this.
- Who is that?
- Zoom in.
- Where is the door?
- Describe this room.

Do not build the full agent yet.

Return structured speech input to the Agent layer.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-11-complete).
```

---

# 13. PHASE 12 — Offline Text-to-Speech

## Goal

Speak responses through the glasses/earphones.

Potential technologies:

- Android TTS for initial prototype;
- Piper/sherpa-onnx or another offline neural TTS system for deeper offline control.

## Prompt

```text
Build Phase 12: Offline Text-to-Speech.

Confirm Definition of Ready: Phase 11 Complete per Phase 11 HANDOFF.md.

Create a TextToSpeechTool.

Requirements:
- offline speech;
- configurable voice;
- configurable language (French and English at minimum);
- speed;
- volume;
- interruption;
- priority;
- queue management.

The output must be routable to:
1. phone speaker for testing;
2. Bluetooth audio;
3. supported glasses audio.

Create tests for queueing and emergency interruption.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-12-complete).
```

---

# 14. PHASE 13 — Vision Aid Agent

## Goal

Turn individual tools into one agent.

Architecture:

```text
User
 ↓
Speech
 ↓
Vision Aid Agent
 ↓
Tool selection
 ↓
Tool execution
 ↓
Context
 ↓
Response
 ↓
TTS
```

## Prompt

```text
Build Phase 13: Vision Aid Agent.

Confirm Definition of Ready: Phase 12 Complete per Phase 12 HANDOFF.md; all tool interfaces from Phase 2 onward exist and are wired up.

Create the central VisionAidAgent that can interpret a user's request and choose the correct local tool.

Examples:

"What is in front of me?"
-> ObjectDetection + WorldModel

"Who is that?"
-> FaceRecognition

"Read this."
-> OCR/DocumentTool

"How much is this?"
-> CurrencyRecognition

"Where is the door?"
-> WorldModel/ObjectDetection

"Describe this room."
-> SceneUnderstanding/VLM if available

The agent must:
- prefer offline tools;
- avoid unnecessary model execution;
- return concise responses;
- report uncertainty;
- never invent observations;
- keep tool calls auditable through local logs.

Do not implement traffic safety decisions yet.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-13-complete).
```

---

# 15. PHASE 14 — Relevance and Context Engine

## Goal

Prevent information overload.

The system should decide what is important.

Example:

```text
Pole 1m ahead -> speak
Tree 20m away -> ignore
Car crossing path -> speak
Bottle 10m away -> ignore
```

## Prompt

```text
Build Phase 14: Relevance and Context Engine.

Confirm Definition of Ready: Phase 13 Complete per Phase 13 HANDOFF.md.

Create a system that scores observations based on:
- distance;
- user's direction;
- walking path;
- movement;
- danger;
- confidence;
- user request;
- current mode;
- recent announcements.

Create priority levels:
1 background
2 useful
3 important
4 warning
5 emergency

The engine must prevent repetitive announcements.

Example:
If a pole remains 1 meter ahead for several seconds, do not repeat the same warning every frame.

Write deterministic tests for priority and deduplication.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-14-complete).
```

---

# 16. PHASE 15 — Navigation

## Goal

Use phone GPS + maps + visual context.

Potential ecosystem:

- OpenStreetMap;
- offline map data;
- suitable routing engine.

## Prompt

```text
Build Phase 15: Offline Navigation.

Confirm Definition of Ready: Phase 14 Complete per Phase 14 HANDOFF.md.

Create NavigationTool supporting:
- current position;
- destination;
- route;
- turn instructions;
- distance;
- offline map data where available.

Integrate GPS from the phone.

Do not depend on Internet for basic navigation.

Keep map/routing providers behind interfaces so they can be replaced.

Do not yet claim pedestrian navigation is safe in every environment.

Create simulation tests for routes and turns.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-15-complete).
```

---

# 17. PHASE 16 — Obstacle Avoidance

**This phase is safety-critical (Section 0.4). Prioritize false-negative analysis and conservative warnings.**

## Goal

Combine the World Model with walking direction.

The system should detect:

- poles
- walls
- people
- vehicles
- stairs
- drop-offs
- objects on the ground

## Prompt

```text
Build Phase 16: Obstacle Assistance.

Confirm Definition of Ready: Phase 15 Complete per Phase 15 HANDOFF.md.

This phase is safety-critical. When uncertain whether an object is in the user's path, prefer issuing a warning over staying silent.

Use:
- object detection;
- tracking;
- depth;
- spatial world model;
- user movement/orientation.

Determine whether an object is likely to be in the user's path.

Generate structured warnings such as:
- obstacle ahead;
- move left;
- move right;
- stop;
- step down.

Do not build traffic crossing decisions yet.

Prioritize false-negative analysis and conservative warnings.

Create recorded-scene tests.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-16-complete). Record false-negative/false-positive rates explicitly in VERIFICATION.md.
```

---

# 18. PHASE 17 — Traffic Safety Engine

**This phase is safety-critical (Section 0.4).**

## Goal

Create a separate safety-critical subsystem.

Pipeline:

```text
Cameras
↓
vehicle detection
↓
tracking
↓
depth
↓
motion estimation
↓
trajectory
↓
crossing zone
↓
risk
↓
warning
```

## Prompt

```text
Build Phase 17: Traffic Safety Engine.

Confirm Definition of Ready: Phase 16 Complete per Phase 16 HANDOFF.md.

This is safety-critical software.

Do NOT use a general-purpose LLM as the sole decision-maker.

Implement a dedicated deterministic safety pipeline using:
- vehicle detection;
- pedestrian detection;
- tracking;
- depth;
- motion estimation;
- trajectory estimation;
- configurable crossing zones;
- multi-frame confirmation.

Output states:
SAFE_CANDIDATE
CAUTION
DANGER
STOP
UNKNOWN

When uncertain, prefer UNKNOWN or CAUTION rather than SAFE.

Never claim that a road is guaranteed safe to cross.

Every decision must include:
- timestamp;
- detected objects;
- confidence;
- estimated distance;
- estimated motion;
- reason for the state.

Create simulation tests before any real-world traffic testing.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-17-complete). Do not tag this phase complete until VERIFICATION.md shows simulation tests passed for every required scenario.
```

---

# 19. PHASE 18 — Emergency Interruption

**This phase is safety-critical (Section 0.4).**

## Goal

Safety messages must override ordinary conversation.

Example:

User:
"What color is this shirt?"

System:
"STOP. Vehicle approaching."

## Prompt

```text
Build Phase 18: Emergency Event Manager.

Confirm Definition of Ready: Phase 17 Complete per Phase 17 HANDOFF.md.

Create a priority-based event system.

Priority:
NORMAL
IMPORTANT
WARNING
CRITICAL
EMERGENCY

Emergency events must interrupt:
- normal AI responses;
- document reading;
- navigation speech;
- scene descriptions.

After the emergency ends, the system may resume the previous task when appropriate.

Create tests proving emergency events always take priority over normal speech.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-18-complete).
```

---

# 20. PHASE 19 — Glasses Integration

## Goal

Connect the actual Vuzix or Rokid device.

Before coding:

1. Choose one platform.
2. Obtain official SDK.
3. Identify camera APIs.
4. Identify microphone APIs.
5. Identify speaker/audio APIs.
6. Identify display APIs.
7. Identify IMU/sensor APIs.
8. Identify Bluetooth/Wi-Fi capabilities.
9. Confirm licensing and developer restrictions.

Do not assume features.

## Prompt

```text
Build Phase 19: Smart Glasses Integration.

Confirm Definition of Ready: Phase 18 Complete per Phase 18 HANDOFF.md.

Target glasses:
[INSERT EXACT MODEL HERE]

I will provide the official SDK/documentation.

Before writing code:
1. inspect the SDK;
2. list available APIs;
3. identify camera capabilities;
4. identify microphone/audio capabilities;
5. identify display capabilities;
6. identify IMU/sensor capabilities;
7. identify Bluetooth/Wi-Fi communication;
8. identify phone-to-glasses communication;
9. identify limitations.

Do not invent SDK APIs.

Create a GlassesDevice interface so the rest of Vision Aid does not depend directly on the vendor.

Implement only verified SDK capabilities.

First milestone:
- connect phone to glasses;
- detect connection state;
- receive supported sensor/camera data;
- send a test audio message;
- send a test display message where supported.

Do not modify hardware yet.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-19-complete).
```

---

# 21. PHASE 20 — Glasses Audio

## Goal

Make the glasses/earphones the primary communication channel.

Test:

- microphone quality;
- speech latency;
- audio clarity;
- noise;
- volume;
- interruption.

## Prompt

```text
Build Phase 20: Glasses Audio Integration.

Confirm Definition of Ready: Phase 19 Complete per Phase 19 HANDOFF.md.

Route Vision Aid speech output through the supported glasses/earphone audio system.

Requirements:
- low latency;
- configurable volume;
- priority handling;
- emergency interruption;
- Bluetooth reconnect;
- fallback to phone audio;
- connection-loss handling.

The application must clearly tell the user when the glasses audio connection is unavailable.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-20-complete).
```

---

# 22. PHASE 21 — Local Memory

## Goal

Remember user-approved information.

Examples:

```text
David -> known person
Home -> saved location
Speech speed -> preference
```

## Prompt

```text
Build Phase 21: Local Memory.

Confirm Definition of Ready: Phase 20 Complete per Phase 20 HANDOFF.md.

Implement encrypted local storage for:
- user preferences;
- known people;
- saved places;
- system settings;
- optional user-defined labels.

Requirements:
- local-first;
- encrypted sensitive information;
- explicit user control;
- delete/reset functionality;
- no automatic cloud upload;
- audit-safe local logging.

Do not store unnecessary personal data.

Create memory APIs for the AI agent.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-21-complete).
```

---

# 23. PHASE 22 — Internet Enhancement

## Goal

Internet adds capabilities without becoming a dependency.

Architecture:

```text
Internet?
 ├── No -> local AI
 └── Yes -> optional advanced services
```

Possible online features:

- advanced LLM
- advanced VLM
- live maps
- translation
- cloud document processing
- model updates
- optional remote assistance

## Prompt

```text
Build Phase 22: Optional Internet Layer.

Confirm Definition of Ready: Phase 21 Complete per Phase 21 HANDOFF.md.

Create a network capability manager.

Requirements:
- detect connectivity;
- keep essential features offline;
- only use online services when explicitly permitted;
- provide graceful fallback;
- prevent accidental upload of private camera/audio/face data;
- expose online/offline state to the agent;
- read any online-service credentials from the secrets/config layer established in Phase 1 — never hard-code.

Create service interfaces for:
- advanced AI;
- online maps;
- translation;
- model updates.

Do not hard-code an external provider.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-22-complete).
```

---

# 24. PHASE 23 — Vision-Language Model

## Goal

Add advanced scene understanding.

Use for:

- room description;
- table description;
- general questions;
- contextual interpretation.

Do not let a general VLM be the sole safety engine.

## Prompt

```text
Build Phase 23: Optional Local Vision-Language Model.

Confirm Definition of Ready: Phase 22 Complete per Phase 22 HANDOFF.md.

Evaluate a small mobile-capable VLM that can run offline on the target phone.

Use it for:
- scene description;
- visual question answering;
- general environmental interpretation.

Do not use the VLM as the sole authority for:
- road crossing;
- collision warnings;
- emergency safety decisions.

Create a VLMTool interface.

Implement model loading only after confirming:
- device compatibility;
- RAM requirements;
- latency;
- license;
- offline capability.

Record the model in MODEL_REGISTRY.md with version and a rollback plan (previous known-good model/version to revert to if this one regresses).

If the model is too heavy, provide a graceful fallback.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-23-complete).
```

---

# 25. PHASE 24 — Situation Engine

## Goal

Combine all perception into situational intelligence.

Example:

```text
User walking
+
pole 1m ahead
+
pole in path
+
stationary
=
"Pole one metre ahead. Move slightly left."
```

## Prompt

```text
Build Phase 24: Situation Engine.

Confirm Definition of Ready: Phase 23 Complete per Phase 23 HANDOFF.md.

Combine:
- WorldModel;
- user movement;
- object tracking;
- depth;
- navigation;
- current user request;
- safety state;
- recent events.

The Situation Engine should determine:
- what matters now;
- what can be ignored;
- what should be spoken;
- what should interrupt;
- what requires confirmation.

Do not make unsupported assumptions.

Every spoken recommendation should be traceable to structured observations.

Create scenario tests:
1. normal walking;
2. pole ahead;
3. stairs;
4. person approaching;
5. vehicle approaching;
6. document reading;
7. known person;
8. unknown environment.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-24-complete).
```

---

# 26. PHASE 25 — Power Optimization

## Goal

Make the phone/glasses system practical.

Do not run every model continuously.

Example:

```text
Always:
- lightweight perception
- tracking
- voice wake detection

On demand:
- OCR
- face recognition
- currency
- VLM
- document analysis
```

## Prompt

```text
Build Phase 25: AI Power Optimization.

Confirm Definition of Ready: Phase 24 Complete per Phase 24 HANDOFF.md.

Profile the application on the actual target phone.

Measure:
- CPU;
- GPU/NPU;
- RAM;
- battery drain;
- temperature;
- inference latency;
- camera FPS.

Implement:
- model activation policies;
- frame skipping;
- dynamic inference frequency;
- model unloading;
- low-power mode;
- thermal safeguards.

Do not sacrifice critical safety perception (Phases 5, 16, 17, 18) merely to save power.

Create a performance report before and after optimization.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-25-complete).
```

---

# 27. PHASE 26 — Model Optimization

## Goal

Make AI models smaller and faster.

Investigate:

- quantization;
- hardware acceleration;
- LiteRT;
- ONNX Runtime;
- vendor NPU acceleration;
- smaller models.

## Prompt

```text
Build Phase 26: Mobile Model Optimization.

Confirm Definition of Ready: Phase 25 Complete per Phase 25 HANDOFF.md.

Audit every AI model.

For each model record in MODEL_REGISTRY.md:
- model name and version;
- model size;
- RAM use;
- latency;
- accuracy;
- license;
- runtime;
- accelerator;
- battery impact;
- rollback plan (previous known-good version and how to revert if an update regresses accuracy).

Evaluate quantized and smaller alternatives.

Do not replace a model solely because it is smaller if accuracy becomes unsafe, especially for the safety-critical phases (5, 16, 17, 18).

Follow the Standard Phase Closing Protocol before finishing (git tag phase-26-complete).
```

---

# 28. PHASE 27 — Hardware Gap Analysis

## Goal

Now determine what the commercial glasses cannot provide.

Create:

```text
Required capability
Actual capability
Gap
Possible solution
Cost
```

Potential missing hardware:

- second camera;
- better stereo camera;
- additional IMU;
- depth sensor;
- external battery;
- custom connector;
- external compute;
- improved microphone.

## Prompt

```text
Build Phase 27: Hardware Gap Analysis.

Confirm Definition of Ready: Phase 26 Complete per Phase 26 HANDOFF.md.

Analyze the current Vision Aid software requirements against the exact smart-glasses model.

Create HARDWARE_GAP_ANALYSIS.md.

For each requirement list:
- required capability;
- existing glasses capability;
- SDK/API availability;
- limitation;
- workaround;
- proposed hardware modification;
- estimated complexity;
- estimated cost.

Do not recommend opening or modifying the glasses unless the limitation has been demonstrated through software/API testing.

Prioritize the cheapest solution.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-27-complete).
```

---

# 29. PHASE 28 — Wearable Prototype

## Goal

Make the entire system wearable.

Initial arrangement:

```text
Glasses
   ↕
Bluetooth/Wi-Fi
   ↕
Phone in pocket/waist pouch
```

The user should not need to operate the phone during normal use.

## Prompt

```text
Build Phase 28: Wearable Prototype Integration.

Confirm Definition of Ready: Phase 27 Complete per Phase 27 HANDOFF.md.

Create a complete user flow:

1. User puts on glasses.
2. User starts Vision Aid.
3. Glasses connect automatically.
4. System checks camera/audio/sensor availability.
5. Offline AI initializes.
6. System announces readiness.
7. User can operate entirely by voice.
8. Emergency events can interrupt normal operation.
9. System gracefully handles disconnection.
10. User can shut down/reset safely.

Create an end-to-end test checklist.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-28-complete).
```

---

# 30. PHASE 29 — Real-World Testing

## Goal

Test outside controlled development environments.

### Indoor

- rooms
- corridors
- stairs
- doors
- furniture
- kitchens

### Outdoor

- sidewalks
- markets
- shops
- roads
- buildings

### Difficult conditions

- darkness
- bright sunlight
- rain
- crowds
- traffic
- noisy environments

## Prompt

```text
Confirm Definition of Ready: Phase 28 Complete per Phase 28 HANDOFF.md.

Create a formal Vision Aid field-testing framework.

Create test cases for:
- obstacle detection;
- distance;
- tracking;
- OCR;
- face recognition;
- currency;
- navigation;
- audio;
- battery;
- connectivity;
- low-light conditions;
- crowded environments.

For every test record:
- environment;
- hardware;
- model versions;
- expected result;
- actual result;
- false positive;
- false negative;
- latency;
- user impact;
- severity.

Do not modify production code automatically from test results.
Generate a prioritized defect list.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-29-complete).
```

---

# 31. PHASE 30 — Blind and Low-Vision User Testing

## Goal

Test the product with actual users.

Measure:

- usability;
- comfort;
- speech overload;
- latency;
- missed obstacles;
- false warnings;
- navigation usefulness;
- OCR;
- recognition;
- battery;
- confidence.

## What I do

**Me:**

- recruit testers;
- obtain informed consent;
- conduct sessions safely;
- collect feedback;
- document failures.

**AI:**

- organize results;
- identify patterns;
- prioritize bugs;
- generate reports.

## Prompt

```text
Confirm Definition of Ready: Phase 29 Complete per Phase 29 HANDOFF.md.

Analyze the attached Vision Aid user-testing results.

Do not invent data.

Group findings into:
1. Safety
2. Accuracy
3. Usability
4. Audio
5. Latency
6. Battery
7. Comfort
8. Privacy
9. Feature requests

Rank issues by severity and frequency.

Create a prioritized development backlog.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-30-complete).
```

---

# 32. PHASE 31 — Safety Validation

**This phase validates the safety-critical scope from Section 0.4.**

## Goal

Before high-risk use, validate safety behavior.

Especially road environments.

Test:

- one car;
- multiple cars;
- motorcycles;
- bicycles;
- parked vehicles;
- turning vehicles;
- vehicles approaching from behind;
- rain;
- darkness;
- intersections;
- divided roads;
- occlusion;
- unusual traffic.

## Prompt

```text
Confirm Definition of Ready: Phase 30 Complete per Phase 30 HANDOFF.md.

Create a formal safety-validation plan for Vision Aid's depth, obstacle, and traffic systems (Phases 5, 16, 17, 18).

Do not claim the system is safe based on model accuracy alone.

Define:
- scenarios;
- environmental conditions;
- expected behavior;
- unacceptable failures;
- false-negative thresholds;
- false-positive thresholds;
- test repetitions;
- logging requirements;
- stop conditions.

Treat road-crossing assistance as safety-critical.

The system must have an UNKNOWN state and must not issue a confident "safe to cross" message when evidence is insufficient.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-31-complete). This tag should not be applied unless a qualified human (you) has reviewed and signed off in VERIFICATION.md.
```

---

# 33. PHASE 32 — Custom Hardware Decision

## Goal

Only now decide whether custom electronics are necessary.

Possible final architecture:

```text
Custom glasses
      ↓
lightweight local processing
      ↓
phone
      ↓
heavy processing / Internet
```

or:

```text
Custom glasses
      ↓
embedded NPU
      ↓
phone
```

## Prompt

```text
Confirm Definition of Ready: Phase 31 Complete and signed off per Phase 31 HANDOFF.md.

Analyze all Vision Aid performance and hardware data.

Determine which functions should remain on:
1. glasses;
2. phone;
3. optional external compute;
4. cloud.

Optimize for:
- safety;
- battery;
- weight;
- latency;
- privacy;
- cost;
- reliability.

Do not recommend custom hardware unless the existing platform cannot meet a demonstrated requirement.

Produce FINAL_COMPUTE_ARCHITECTURE.md.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-32-complete).
```

---

# 34. PHASE 33 — Custom Glasses Electronics

Only after proving the need.

Potential components:

- synchronized RGB cameras;
- optional IR/low-light camera;
- microphone array;
- speakers/open-ear audio;
- IMU;
- Bluetooth/Wi-Fi;
- embedded NPU if justified;
- memory;
- battery;
- charging;
- optional display.

At this stage you need:

- electronics engineer;
- PCB designer;
- mechanical engineer;
- industrial designer;
- embedded engineer.

AI can help with engineering documentation and code, but physical engineering and certification require qualified professionals and real measurements.

*(No standard phase prompt — this phase is primarily human-led with AI assisting on documentation.)*

---

# 35. PHASE 34 — Production Software

Add:

- secure updates;
- crash diagnostics;
- model updates;
- device recovery;
- permissions;
- privacy controls;
- encrypted storage;
- factory reset;
- accessibility settings;
- offline recovery;
- model/version management.

## Prompt

```text
Confirm Definition of Ready: Phase 32 Complete per Phase 32 HANDOFF.md (and Phase 33 hardware decisions if applicable).

Prepare Vision Aid for production software engineering.

Audit:
- security;
- privacy;
- permissions;
- encryption;
- offline behavior;
- update system;
- crash recovery;
- model versioning;
- logging;
- data retention;
- account/device management;
- secrets/config handling per Section 0.3.

Do not expose private camera, audio, face or location data unnecessarily.

Create a production-readiness checklist.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-34-complete).
```

---

# 36. PHASE 35 — Regulatory and Certification Preparation

Before commercial launch investigate the exact requirements for intended markets.

Potential areas:

- radio certification;
- Bluetooth/Wi-Fi compliance;
- battery safety;
- electronics safety;
- privacy;
- biometric data;
- accessibility;
- medical/assistive-device classification;
- product liability;
- consumer protection.

AI should help research and organize requirements, but qualified regulatory/legal professionals should validate the final interpretation.

## Prompt

```text
Confirm Definition of Ready: Phase 34 Complete per Phase 34 HANDOFF.md.

Create a regulatory research checklist for Vision Aid Glasses.

Target markets:
[INSERT COUNTRIES/REGIONS — e.g. Cameroon, and any export markets]

Research and organize requirements related to:
- wearable electronics;
- radio equipment;
- batteries;
- Bluetooth/Wi-Fi;
- privacy;
- biometric data;
- accessibility;
- assistive technology;
- medical-device classification where applicable;
- software updates;
- product liability.

Do not provide legal certainty.
Clearly distinguish official requirements from recommendations.

Cite official sources.

Follow the Standard Phase Closing Protocol before finishing (git tag phase-35-complete).
```

---

# 37. PHASE 36 — Production Prototype

Final system:

```text
                VISION AID GLASSES
                       │
          ┌────────────┼────────────┐
          │            │            │
       CAMERAS        MIC        AUDIO
          │            │            │
          └────────────┼────────────┘
                       │
                Bluetooth/Wi-Fi
                       │
                       ▼
                ┌─────────────┐
                │    PHONE    │
                │             │
                │ AI Agent    │
                │ Vision      │
                │ Depth       │
                │ Tracking    │
                │ OCR         │
                │ Faces       │
                │ Currency    │
                │ Navigation  │
                │ Safety      │
                │ Memory      │
                └──────┬──────┘
                       │
                 Optional Internet
                       │
                       ▼
                Advanced Services
```

---

# 38. Final AI Architecture

The mature system should look approximately like this:

```text
                         USER
                           │
                     Voice / movement
                           │
                           ▼
                    SMART GLASSES
                           │
               Camera / Mic / Sensors
                           │
                           ▼
                    PHONE DEVICE
                           │
             ┌─────────────┴─────────────┐
             │                           │
       PERCEPTION LAYER             AUDIO LAYER
             │                           │
     ┌───────┼────────┐            ASR / TTS
     │       │        │
   Object   Face     OCR
   Depth    Track    Currency
     │       │        │
     └───────┼────────┘
             ▼
         WORLD MODEL
             │
             ▼
      SITUATION ENGINE
             │
       ┌─────┴──────┐
       │            │
    CONTEXT       SAFETY
       │            │
       └─────┬──────┘
             ▼
         AI AGENT
             │
             ▼
      PRIORITY MANAGER
             │
             ▼
          RESPONSE
             │
             ▼
        GLASSES AUDIO
```

---

# 39. Development Milestones

## V1 — SEEING

Deliver:

- camera
- object detection
- tracking
- basic depth
- spatial model

User can ask:

> "What's in front of me?"

---

## V2 — UNDERSTANDING

Deliver:

- faces
- OCR
- documents
- currency
- magnification
- scene understanding

User can ask:

> "Who is that?"

> "Read this."

> "How much money is this?"

---

## V3 — ASSISTING

Deliver:

- voice agent
- navigation
- contextual responses
- obstacle assistance
- local memory

User can say:

> "Take me to the pharmacy."

---

## V4 — PROTECTING

Deliver:

- traffic detection
- tracking
- trajectory estimation
- safety engine
- emergency interruption

This phase requires extensive validation.

---

## V5 — PRODUCT

Deliver:

- glasses integration
- offline-first operation
- optimized models
- battery optimization
- user testing
- production hardware
- privacy/security
- certification preparation

---

# 40. What the Human Does vs What AI Does

## YOU

You are responsible for:

- product vision;
- decisions;
- hardware purchases;
- choosing the glasses;
- choosing the phone;
- physical assembly;
- SDK access;
- real-world testing;
- user recruitment;
- safety testing;
- business decisions;
- regulatory decisions;
- approving releases;
- signing off VERIFICATION.md for safety-critical phases (5, 16, 17, 18, 31).

## AI

AI can help with:

- architecture;
- Android code;
- model integration;
- APIs;
- testing;
- documentation;
- debugging;
- optimization;
- data-processing scripts;
- UI;
- model evaluation;
- logs;
- reports;
- technical research;
- maintaining PROGRESS.md, HANDOFF.md, and VERIFICATION.md.

## AI should NOT independently decide:

- that a road is safe;
- that a medical/safety claim is valid;
- that a product is certified;
- that a face belongs to someone when confidence is insufficient;
- that hardware is electrically safe;
- that a component is compatible without documentation;
- that a safety-critical phase (5, 16, 17, 18, 31) is complete without your sign-off in VERIFICATION.md.

---

# 41. Recommended Project Repository

Use a structure like:

```text
vision-aid-glasses/
│
├── README.md
├── PRODUCT_SPEC.md
├── ARCHITECTURE.md
├── ROADMAP.md
├── SAFETY.md
├── PRIVACY.md
├── MODEL_REGISTRY.md
├── CODING_STANDARDS.md
├── PROGRESS.md
├── HANDOFF.md
├── .env.example
│
├── android/
│   └── VisionAid/
│
├── ai/
│   ├── object-detection/
│   ├── tracking/
│   ├── depth/
│   ├── face/
│   ├── ocr/
│   ├── currency/
│   ├── speech/
│   └── tts/
│
├── agent/
│
├── navigation/
│
├── safety/
│
├── glasses/
│   ├── vuzix/
│   └── rokid/
│
├── models/
│
├── tests/
│
├── datasets/
│   └── DATASET_PROVENANCE.md   (source, license, consent for face/currency data)
│
├── hardware/
│
└── docs/
    └── verification/
        ├── phase-0.md
        ├── phase-1.md
        └── ... (or a single VERIFICATION.md — pick one convention)
```

Do not necessarily implement both Vuzix and Rokid integrations simultaneously. Keep vendor-specific code isolated so one can be added later.

---

# 42. Master Prompt for Starting the Project

Use this only when beginning the project repository. After this, use the individual phase prompts above — each of which already includes the Definition of Ready check and Standard Phase Closing Protocol.

```text
You are the lead software architect and senior Android/AI engineer working with me on a project called Vision Aid Glasses.

PRODUCT:
Vision Aid Glasses is an offline-first wearable AI perception and assistance system for blind and low-vision users.

ARCHITECTURE:
- Smart glasses provide cameras, microphones, audio, display where supported and sensors.
- An Android phone provides most AI compute, storage, battery, GPS and connectivity.
- Internet is optional.
- Essential assistive functions must work offline.
- A Jetson or external AI computer is optional and must not be assumed necessary.
- Initial hardware will use an existing developer-oriented smart-glasses platform such as Vuzix or Rokid.
- Hardware modifications will only be considered after software/API limitations are demonstrated.
- ASR/TTS must support French and English at minimum.

CORE PRODUCT PRINCIPLE:
The system must understand what matters to the user rather than describe every visible object.

Examples:
Instead of:
"There is a tree, Toyota, woman, motorcycle, bottle..."

Prefer:
"Pole one metre ahead. Move slightly left."

CORE SYSTEMS:
1. Object detection
2. Object tracking
3. Stereo/depth perception
4. Spatial world model
5. Face recognition
6. OCR
7. Document understanding
8. Currency recognition
9. Magnification
10. Offline speech recognition
11. Offline text-to-speech
12. AI agent
13. Context/relevance engine
14. Navigation
15. Obstacle assistance
16. Traffic safety engine
17. Emergency interruption
18. Local encrypted memory
19. Optional Internet services
20. Optional local VLM
21. Glasses integration
22. Power/model optimization

DEVELOPMENT RULE:
Build one phase at a time.

For every phase:
- read PROGRESS.md and HANDOFF.md and confirm Definition of Ready before starting;
- inspect existing code;
- implement only the requested phase;
- do not rewrite unrelated functionality;
- write tests;
- follow CODING_STANDARDS.md;
- document the implementation;
- maintain modular interfaces;
- preserve offline functionality;
- do not invent SDK APIs;
- do not claim unsupported hardware capabilities;
- clearly identify limitations;
- update PROGRESS.md, HANDOFF.md, and VERIFICATION.md, then git commit and tag phase-N-complete;
- stop after completing the requested phase and report what changed.

CONTINUATION:
Once a phase is complete, I will simply say "build next phase" or "continue with the next." Before doing anything, read PROGRESS.md and HANDOFF.md to determine current state and the next incomplete phase.

If I start a new chat in this same project, I will upload only: PROGRESS.md, HANDOFF.md, VERIFICATION.md, CODING_STANDARDS.md, and ARCHITECTURE.md. Treat these as the full current state of the project unless you need to inspect specific source files, in which case ask.

SAFETY:
Depth (Phase 5), Obstacle Avoidance (Phase 16), Traffic Safety (Phase 17), and Emergency Interruption (Phase 18) are safety-critical.
Never use a general-purpose LLM as the sole safety decision-maker.
The system must support UNKNOWN/UNCERTAIN states.
When uncertain, prefer conservative behavior.
Do not claim that the system guarantees safe road crossing.
Do not mark a safety-critical phase complete without my explicit sign-off in VERIFICATION.md.

PRIVACY:
Face data, voice, images, location and personal memory should remain local by default.
No unnecessary cloud upload.
Document dataset provenance and consent for any face or currency training data (datasets/DATASET_PROVENANCE.md).

START:
Do not build the entire application now.

First inspect the repository and produce:
1. current project structure;
2. recommended architecture;
3. dependencies;
4. risks;
5. Phase 1 implementation plan.

Then wait for approval before implementing Phase 1.
```

---

# 43. The Golden Rule

The project should evolve like this:

```text
          BUILD
            ↓
          TEST
            ↓
        MEASURE
            ↓
         REVIEW
            ↓
          FIX
            ↓
       CHECKPOINT (git tag phase-N-complete)
            ↓
   UPDATE PROGRESS/HANDOFF/VERIFICATION
            ↓
      NEXT PHASE (triggered by "build next phase")
```

**Never:**

```text
Prompt AI once
      ↓
Build everything
      ↓
Hope it works
```

The first goal is **not to build the final glasses**.

The first goal is to create a reliable **Vision Aid software brain** that can see, understand, reason and communicate.

Once that brain works reliably on the phone, connecting it to Vuzix/Rokid becomes an integration problem. Only after that should you decide what hardware needs to be changed.

---

# 44. Immediate Starting Point

The first practical sequence is:

```text
1. Choose Android phone
          ↓
2. Create GitHub repository
          ↓
3. Phase 0 specification
          ↓
4. Phase 1 Android foundation (includes CODING_STANDARDS.md, secrets pattern)
          ↓
5. Phase 2 AI tool architecture
          ↓
6. Phase 3 object detection
          ↓
7. Phase 4 tracking
          ↓
8. Phase 5 depth (safety-critical)
          ↓
9. Phase 6 spatial world model
          ↓
10. Only then begin selecting/integrating glasses
```

At that point you will already have a functioning **phone-based Vision Aid brain** before spending heavily on custom hardware, with a full paper trail of what was built, tested, and verified at every step.

---

# 45. Quick Reference — Daily Use

**To continue building:** say "build next phase" or "continue with the next." The AI reads `PROGRESS.md` + `HANDOFF.md` and proceeds automatically.

**To resume in a new chat within the same project:** upload `PROGRESS.md`, `HANDOFF.md`, `VERIFICATION.md`, `CODING_STANDARDS.md`, and `ARCHITECTURE.md`, then say "continue with the next phase." Do not re-upload the full source tree unless asked.

**To check status at any time:** ask "what's our current progress?" — the AI should read `PROGRESS.md` and summarize.

**Before approving any safety-critical phase (5, 16, 17, 18, 31) as complete:** review `VERIFICATION.md` yourself and sign off — don't let the AI self-certify these.
