# Vision Aid Glasses — Product Specification (PRODUCT_SPEC.md)

**Phase:** 0 — Product Definition
**Status:** Approved (formalizes decisions in Vision_Aid_Glasses_Master_Roadmap.md)
**Date:** 2026-09-12

---

## 1. Product Vision

Vision Aid Glasses is an **offline-first wearable AI perception and assistance system** for blind and low-vision users. Smart glasses provide the user's eyes, ears, and interface (cameras, microphones, speakers/earphones, display where supported, IMU/sensors). An Android phone, carried separately, performs the bulk of AI inference, storage, battery, and connectivity.

The system does not attempt to describe everything it sees. It determines what matters to the user in the moment and communicates concise, actionable information through audio and, where supported, visual output. Internet access is optional and only ever *enhances* capability — it is never a dependency for core assistive functions.

## 2. Target Users

- **Primary:** Blind and low-vision individuals who want situational awareness, obstacle/safety warnings, reading assistance, and person/object/currency identification without relying on a sighted companion.
- **Secondary:** Low-vision users who retain partial sight and benefit from magnification, contrast enhancement, and text enlargement rather than full audio-first assistance.
- **Geography:** Cameroon (Douala) is the primary target market for initial deployment and testing. The product must remain usable and useful without dependable Internet access, consistent with connectivity conditions in the target market.
- **Out of scope for V1–V3:** Users requiring certified medical-device-grade guidance (e.g., independent mobility replacement for a white cane in all conditions) — see Non-Goals (Section 11) and Safety Limitations (Section 12).

## 3. Low Vision Mode

For users with residual usable vision. Features:

- **Magnification** — digital zoom (2x/4x/8x) of the camera feed, routed to a glasses display where the hardware supports one.
- **Text enlargement** — enlarging recognized text for on-device or glasses-display viewing.
- **OCR** — converting printed/handwritten text in view into digital text for enlargement or read-aloud.
- **Face identification** — recognizing previously enrolled people.
- **Object recognition** — identifying common objects in the visual field.
- **Colors** — identifying the color of a selected object or region.
- **Document reading** — reading structured documents (letters, labels, forms) aloud or displaying enlarged text.

Low Vision Mode assumes the glasses hardware exposes a display; where it does not, magnification degrades gracefully to a phone-side preview mode (see Phase 9 in the roadmap).

## 4. Blind Assistance Mode

For users with little or no usable vision. Audio-first. Features:

- **Obstacle detection** — identifying objects likely to be in the user's walking path.
- **Object identification** — naming and locating relevant objects on request.
- **People identification** — recognizing enrolled people and flagging unknown presence.
- **Currency recognition** — identifying banknote/coin denomination, starting with Cameroon CFA franc (XAF).
- **Document reading** — full audio reading of documents, page by page.
- **Environmental description** — concise scene summaries on request ("Describe this room").
- **Navigation** — turn-by-turn guidance to a destination using phone GPS and offline map data where available.

## 5. Safety Mode

Always-active safety layer, running alongside whichever primary mode is selected. Features:

- **Obstacle warnings** — spoken alerts for poles, walls, furniture, and other static/dynamic obstacles in the user's path.
- **Moving vehicle detection** — detecting vehicles and their approximate trajectory relative to the user.
- **Pedestrian detection** — detecting people who may intersect the user's path.
- **Stairs/drop-off detection** — detecting elevation changes (steps, curbs, drop-offs).
- **Road awareness** — contextual signals about proximity to roads/crossings (not a certified crossing-safety guarantee — see Section 12).
- **Emergency interruption** — safety-critical spoken warnings interrupt any other audio output immediately.

Safety Mode corresponds to the safety-critical roadmap phases (5, 16, 17, 18) and inherits all constraints in Section 0.4 of the roadmap: no general-purpose LLM/VLM may be the sole decision-maker for these functions, and the system must expose deterministic, testable outputs with an explicit confidence/uncertainty state, always preferring `UNKNOWN`/conservative output to a confident-but-wrong result.

## 6. Offline Requirements

The following must function with **zero** Internet connectivity:

- Object detection, tracking, depth estimation, and the spatial world model.
- Obstacle avoidance and safety-critical warnings (Phases 5, 16, 17, 18).
- Face recognition (local enrollment and matching only).
- OCR and document reading.
- Currency recognition.
- Magnification (Low Vision Mode).
- Offline speech recognition (ASR) and text-to-speech (TTS) in French and English.
- Local encrypted memory (known people, saved places, preferences).
- Emergency interruption / priority-based event handling.
- Basic navigation using pre-downloaded/offline map data, where available.

## 7. Online Capabilities (Optional Enhancements Only)

Internet access, when available and explicitly permitted by the user, may enhance — never replace — the following:

- Advanced LLM/VLM-based scene understanding for open-ended questions.
- Live map data, richer routing, and traffic-aware navigation.
- Translation beyond the offline-supported languages.
- Cloud-assisted document processing for complex layouts.
- Model updates delivered over the network.
- Optional remote human assistance (future consideration, not committed to any V1–V4 milestone).

The system must clearly expose online/offline state to the user and to the AI agent, and must never silently degrade a safety-critical function's behavior based on connectivity.

## 8. Privacy Requirements

- Face images/embeddings, voice recordings, camera frames, and location data remain **local by default**. No automatic upload to any cloud service.
- Any online feature (Section 7) that would transmit camera, audio, face, or location data requires **explicit, feature-specific user consent** — not a single blanket permission.
- Face enrollment requires explicit in-app consent captured before storage (roadmap Phase 7).
- Any pretrained model used for face or currency recognition must have documented training-data provenance and a license permitting this use (`datasets/DATASET_PROVENANCE.md`).
- Users must be able to view, delete, and reset all locally stored personal data (known people, saved places, preferences) at any time.
- The system must avoid collecting personal data not needed for a stated feature.
- Local sensitive data (face embeddings, personal memory) must be encrypted at rest.

## 9. Hardware Assumptions

- **Phone:** Not yet selected — **TBD**. The architecture must not assume a specific chipset beyond "a reasonably current Android phone capable of on-device ML inference"; exact minimum specs (RAM, NPU/GPU, Android version) will be finalized once profiling data exists (roadmap Phase 25).
- **Glasses:** No specific model selected yet — **TBD**. The roadmap recommends starting with an existing developer-oriented platform (e.g., Vuzix or Rokid) rather than custom hardware. Assumed capabilities (cameras, microphone, speaker/audio, optional display, IMU, Bluetooth/Wi-Fi) are provisional until an exact model is chosen and its SDK/documentation is inspected (roadmap Phase 19). No glasses capability may be assumed or invented ahead of that inspection.
- **Stereo depth:** Not guaranteed — the system must not assume the selected glasses expose two synchronized cameras (roadmap Phase 5); a hardware capability abstraction is required so depth estimation can fall back to monocular methods.
- **External compute (e.g., Jetson):** Optional, for development/heavy experimentation only. Not a requirement for the shipped product.
- **Connectivity:** Bluetooth and/or Wi-Fi between glasses and phone; optional cellular/Wi-Fi Internet from the phone.

All hardware-dependent capabilities in this spec are marked TBD and must be confirmed against real SDK/API documentation before implementation, per the roadmap's rule against inventing APIs or hardware capabilities.

## 10. Software Requirements

- Android/Kotlin application, modular architecture (core, AI, vision, audio, navigation, safety, memory, documents, currency, glasses modules).
- Voice-first interaction as the primary interface; screen interaction is a secondary/testing affordance only.
- Local encrypted storage for memory and personal data.
- Structured, replaceable AI tool interfaces (Object Detection, Tracking, Depth, Face Recognition, OCR, Currency, Navigation, Document, Safety, Speech Recognition, Text-to-Speech) so individual models can be swapped without rebuilding the application.
- Priority-based audio event system (NORMAL / IMPORTANT / WARNING / CRITICAL / EMERGENCY) with guaranteed emergency interruption.
- Config/secrets layer with no credentials ever committed to source control.
- Structured logging sufficient for debugging and auditability, without over-collecting personal data.
- Low latency and low power consumption as first-class design constraints, not afterthoughts.

## 11. Non-Goals

The following are explicitly **out of scope** for this product line (V1–V5 as currently scoped):

- Replacing a white cane, guide dog, or other certified mobility aid as the user's sole means of independent mobility.
- Serving as a certified medical device or providing medical diagnoses.
- Guaranteeing safe road crossing under any condition — the system provides supporting information only (see Section 12).
- Full custom glasses hardware/electronics design, unless and until Phase 27 (Hardware Gap Analysis) demonstrates the existing platform cannot meet a proven requirement.
- Cloud-based identification of unknown people using external databases or the Internet — the system only recognizes people the user has explicitly enrolled.
- Continuous, unconditional cloud upload of camera/audio/location data.
- Supporting languages beyond French and English at product launch (additional local languages are an explicit future extension point, not a V1–V3 commitment).

## 12. Safety Limitations

- The system is an **assistive aid, not a guarantee of safety**. It must never claim a road is "safe to cross" or that an obstacle-free path is certain.
- Depth (Phase 5), Obstacle Avoidance (Phase 16), Traffic Safety (Phase 17), and Emergency Interruption (Phase 18) are safety-critical and must always expose an explicit `UNKNOWN`/low-confidence state rather than a false precise or confident answer.
- A general-purpose LLM/VLM must never be the sole decision-maker for any safety-critical function.
- No safety-critical phase may be marked complete without human sign-off in `VERIFICATION.md` (roadmap Sections 0.4, 40).
- Environmental conditions (darkness, rain, glare, crowd density, occlusion) can degrade perception accuracy; the product must communicate reduced confidence under such conditions rather than presenting false certainty.
- Users must be informed, at onboarding, that the system is a supplementary aid and does not replace standard orientation-and-mobility training or existing mobility aids.

## 13. MVP / V1 / V2 / V3 (and V4/V5) Feature Boundaries

Per the roadmap's milestone structure:

- **V1 — Seeing (MVP):** Camera input, object detection, object tracking, basic depth (near/medium/far), spatial world model. Enables: *"What's in front of me?"*
- **V2 — Understanding:** Face recognition, OCR/document reading, currency recognition, magnification, local scene understanding. Enables: *"Who is that?"*, *"Read this,"* *"How much money is this?"*
- **V3 — Assisting:** Voice agent, offline navigation, contextual/relevance-scored responses, obstacle assistance, local encrypted memory. Enables: *"Take me to the pharmacy."*
- **V4 — Protecting:** Traffic detection, trajectory estimation, dedicated traffic safety engine, emergency interruption. Requires extensive safety validation (Phase 31) before any real-world traffic use.
- **V5 — Product:** Glasses hardware integration, full offline-first operation, optimized/quantized models, battery optimization, real-world and blind/low-vision user testing, production hardware decisions, security/privacy hardening, regulatory/certification preparation.

No milestone may claim capabilities belonging to a later milestone (e.g., V1 must not imply traffic-safety guarantees, which only arrive at V4 after validation).

## 14. Acceptance Criteria (Phase 0)

- [x] Product vision, target users, and three operating modes (Low Vision, Blind Assistance, Safety) are documented.
- [x] Offline vs. online capability boundaries are explicit and consistent with the roadmap's offline-first principle.
- [x] Privacy requirements, including dataset provenance and consent for biometric data, are documented.
- [x] Hardware assumptions are documented and correctly marked TBD where no phone/glasses model has been selected — no invented hardware capabilities.
- [x] Non-goals and safety limitations are explicit, including the prohibition on LLM/VLM-only safety decisions and the requirement for human sign-off on safety-critical phases.
- [x] MVP/V1–V5 feature boundaries are defined and traceable to the roadmap's milestone section.
- [x] Localization requirements (Section 15) specify French and English as minimum-supported languages with locale passed as a parameter.
- [ ] Reviewed and approved by product owner (human sign-off — see note below).

**Note:** This spec formalizes decisions already stated in `Vision_Aid_Glasses_Master_Roadmap.md`, per your instruction to proceed "as per the documentation." Hardware selections (Section 9) remain open and should be filled in before Phase 19 (Glasses Integration) is attempted; they do not block Phases 1–18, which are hardware-abstracted.

## 15. Localization Requirements

- ASR and TTS must support **French and English at minimum** — Cameroon's official languages — from Phase 0 onward.
- No tool interface may hard-code a language assumption; locale is always passed as an explicit parameter.
- OCR must support French and English text output at minimum (roadmap Phase 8).
- Currency recognition targets **Cameroon CFA franc (XAF)** first (roadmap Phase 10), with the model interface designed so additional currencies can be added later without redesign.
- The architecture must remain open to adding further local languages beyond French/English in future phases, without requiring rework of the core tool interfaces.
