# Vision Aid Glasses — Coding Standards

These conventions apply to every module and every future phase. Any AI agent
or contributor working on this codebase should follow this document; deviate
only with a documented reason recorded in that phase's `HANDOFF.md`.

## Language & Structure

- **Kotlin** for all Android code. No Java files except where an SDK
  strictly requires it (document the exception in `HANDOFF.md` if it ever
  happens).
- One class/interface per file, file name matches the type name.
- Modules communicate only through their public interfaces (the `ai` module's
  tool interfaces, `core`'s logging/config/settings). Never reach into
  another module's internal package directly.
- No module may depend on `app`. Dependencies flow one direction:
  `app` → feature modules → `core`.

## Naming

- Packages: `com.visionaid.<module>[.<subpackage>]`, all lowercase.
- Classes/interfaces/objects: `UpperCamelCase`.
- Functions/properties: `lowerCamelCase`.
- Constants: `UPPER_SNAKE_CASE` inside a `companion object` or top-level
  `const val`.
- Test classes: `<ClassUnderTest>Test`. Test function names use backtick
  descriptive phrases: `` fun `returns UNKNOWN when confidence below threshold`() ``.
- AI tool interfaces end in `Tool` (e.g. `ObjectDetectionTool`). Their
  Android/production implementations are prefixed by technique or vendor
  where relevant (e.g. `LiteRtObjectDetectionTool`); mock implementations
  are prefixed `Mock` (e.g. `MockObjectDetectionTool`).

## Error Handling

- Never silently swallow an exception. At minimum, log it via
  `VisionAidLogger.e(tag, message, throwable)`.
- AI/perception tools never throw for "no detection" or "low confidence" —
  they return a structured result with a confidence/uncertainty field and,
  for safety-critical tools, an explicit `UNKNOWN` state (Section 0.4 of the
  roadmap). Reserve exceptions for genuine failures (model failed to load,
  camera unavailable, I/O error).
- Safety-critical modules (`safety`, and anything feeding Phases 5/16/17/18)
  must fail closed: on any internal error, the module returns `UNKNOWN` /
  issues a conservative warning rather than staying silent or guessing.
- User-facing errors are surfaced through the TTS/priority-event system
  (Phase 12/14/18), not raw stack traces or toasts.

## Logging

- Use `VisionAidLogger` exclusively — never raw `android.util.Log` calls in
  feature modules.
- Never log: raw camera frames, raw audio buffers, face embeddings, exact
  GPS coordinates, or any other content that could reconstruct personal/
  biometric data, at any log level. Log metadata about an event (e.g. "face
  match attempted, confidence=0.82"), not the underlying biometric payload.
- One tag per class (typically the class's simple name as a `private const
  val TAG`).

## Secrets & Configuration

- No credential, API key, or token is ever hard-coded, committed, or logged.
- All config/secret reads go through `com.visionaid.core.config.SecretsConfig`
  (or its successor once Phase 22 extends it). Document every new key in
  `.env.example` at the repo root with a placeholder value.

## Testing

- Every new class with non-trivial logic gets a unit test in the same
  module's `src/test` source set.
- Safety-critical logic (Phases 5, 16, 17, 18) requires tests that explicitly
  assert conservative behavior under uncertainty (e.g. "returns DANGER or
  UNKNOWN, never SAFE, when confidence is below threshold").
- Prefer deterministic, synthetic test fixtures (recorded frames, canned
  detection sequences) over relying on a live camera/model in unit tests.

## Documentation

- Every public interface gets a KDoc comment describing its input contract,
  output contract, and failure modes.
- Any placeholder/stub code (like the Phase 1 module placeholders) must say
  so explicitly in its KDoc, including which future phase replaces it.

## Git

- Commit message format: `Phase N: <short description>`.
- Tag format: `phase-N-complete`, applied only after `VERIFICATION.md` shows
  PASSED or PASSED WITH NOTES for that phase (and human sign-off for
  safety-critical phases 5, 16, 17, 18, 31).
