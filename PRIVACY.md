# Vision Aid Glasses — Privacy: Face Data

This is the first entry in what the roadmap's repository layout (Section 41)
calls `PRIVACY.md` — starting with face/biometric data because Phase 7 is
the first phase to touch it. Future phases handling other personal data
(location in Phase 15, general local memory in Phase 21) should extend this
file rather than create parallel privacy documents.

## What is collected

- A face **embedding** (a numeric vector derived from a face image) for each
  person the user explicitly chooses to enroll.
- The person's display name, as typed/spoken by the user.

## What is NOT collected

- The original face photo/image is never stored — only the derived
  embedding. (Practically moot at this phase anyway: no embedding model is
  currently integrated — see `MODEL_REGISTRY.md`'s "Face Recognition /
  Identity Embedding" section for why.)
- No data about faces detected but not enrolled — an unrecognized face is
  processed transiently during `recognize()` and discarded; nothing about
  it is stored.
- No data ever leaves the device. There is no network call anywhere in the
  face detection/recognition/storage code path.

## Consent

Enrollment (`FaceRecognitionTool.enroll()`) requires an explicit
`consentConfirmed: Boolean` parameter — this is enforced **structurally in
the interface**, not just documented: every implementation (including the
Phase 2 mock) rejects enrollment with a `ToolError.InvalidInput` failure if
this is false. The actual consent-capture UI (the screen/dialog that sets
this to true) is an application-layer concern for whichever phase builds
the enrollment flow's UI — this interface guarantees the *storage* layer
never proceeds without it, regardless of what the UI looks like.

## Storage

`EncryptedFaceProfileStore` (`vision/face/`) uses Jetpack Security's
`EncryptedSharedPreferences`: AES-256-GCM for values, AES-256-SIV for keys,
with the master key held in the Android Keystore (hardware-backed where the
device supports it). No custom cryptography, no plaintext fallback path.

## User control

Every operation the roadmap's Phase 7 spec requires is implemented:

- **Enroll** — `enroll(frame, personName, consentConfirmed)`.
- **Rename** — `rename(personId, newName)`.
- **Remove** — `remove(personId)`, deletes the stored profile entirely.
- **Temporarily disable recognition** — `setEnabled(false)`; while disabled,
  `recognize()`/`enroll()` fail rather than silently no-op, so the caller
  always knows the tool is off.

There is currently no "delete everything" bulk operation exposed through
the `FaceRecognitionTool` interface itself, though
`EncryptedFaceProfileStore.clearAll()` exists at the storage layer — a
future phase (likely Phase 21, Local Memory, which covers app-wide data
reset) should decide whether to expose bulk deletion through this interface
or through a general settings/reset screen instead, rather than this phase
guessing at that UX decision.

## Known gap carried forward

Recognition currently cannot produce a real result — `availability()`
reports `UNAVAILABLE_MODEL_NOT_LOADED` because no face-embedding model has
been integrated (see `MODEL_REGISTRY.md` for the full reasoning: available
pretrained options have documented consent problems in their training
data). This is a deliberate compliance decision, not a bug. Enrollment will
also fail until this is resolved, since generating an embedding is a
prerequisite for storing a profile.
