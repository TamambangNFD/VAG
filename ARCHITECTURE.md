# Vision Aid Glasses — Architecture (Phase 1 baseline)

This document describes the Phase 1 Android foundation and the structural
decisions that later phases must build on. It will be updated as later
phases add real capability; this version reflects only what exists after
Phase 1.

## Module Graph

```
                app
                 │
  ┌────┬────┬────┼────┬────────┬─────────┬───────────┬──────────┐
  ai vision audio │  navigation memory documents currency glasses
                 core
```

- `app` — the Android application shell (Application class, MainActivity,
  manifest/permissions). Depends on every feature module. No feature module
  depends on `app`.
- `core` — cross-cutting foundation: logging (`VisionAidLogger`),
  secrets/config access (`SecretsConfig`), and shared settings data classes
  (`AppSettings`, `SupportedLanguage`). Every other module depends on `core`
  and nothing else at this phase.
- `ai`, `vision`, `audio`, `navigation`, `memory`, `documents`, `currency`,
  `glasses` — feature module boundaries established now so later phases add
  real code without restructuring the project. Each currently contains only
  a documented placeholder object and a trivial test proving the module
  builds and is wired into `settings.gradle.kts` correctly.
- `safety` — same as above, but flagged as the eventual home of
  safety-critical logic (Phases 5, 16, 17, 18). It already declares its
  Phase-2-onward dependencies on `vision` and `ai` so the dependency
  direction is fixed before any safety logic is written.

## Why modules exist before their features do

The roadmap's Phase 1 goal is the *foundation*, not the features. Creating
empty-but-real Gradle modules now (rather than a single monolithic `app`
module) means:

1. Later phases can be reviewed and tested in isolation without touching
   unrelated code — directly satisfying the roadmap's "never rewrite
   unrelated working code" rule.
2. Dependency direction is enforced by the build system, not convention:
   Gradle will fail to compile if, say, `vision` tries to depend on `app`.
3. The safety-critical module (`safety`) is structurally isolated from
   day one, making it easier to audit in Phase 31 (Safety Validation).

## Secrets & Configuration Pattern (Section 0.3)

- `.env.example` (repo root) and `android/VisionAid/local.properties.example`
  document every config key a phase will need, with placeholder values only.
- Real values go in a git-ignored `.env` or `local.properties` — never
  committed (`.gitignore` enforces this).
- `com.visionaid.core.config.SecretsConfig` is the single code path for
  reading configuration/secrets. Phase 1 only reads safe, non-secret flags
  (e.g. `VISIONAID_OFFLINE_ONLY`) because no online service exists yet.
  Phase 22 (Optional Internet Layer) will extend this object to read real
  credentials for online services — it must not introduce a second, parallel
  config mechanism.

## Permissions (Phase 1 baseline)

Declared in `AndroidManifest.xml`, each tied to a named future phase:

| Permission | Justified by |
|---|---|
| `CAMERA` | Object detection, OCR, face recognition, currency (Phases 3, 7, 8, 10) |
| `RECORD_AUDIO` | Offline speech recognition (Phase 11) |
| `BLUETOOTH_CONNECT` / `BLUETOOTH_SCAN` | Glasses communication (Phase 19-20) |
| `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` | Navigation (Phase 15) |
| `INTERNET` / `ACCESS_NETWORK_STATE` | Optional enhancement layer only (Phase 22) — never required for offline core functions |

No permission is requested that isn't tied to a specific, named future
capability, per the roadmap's "avoid collecting unnecessary personal data"
principle.

## Offline-First Enforcement

`AppSettings.offlineOnly` defaults to `true`. No code path in Phase 1 makes a
network call. This default must not be silently changed by any future phase
without an explicit, user-facing toggle (Phase 22).

## Logging

`VisionAidLogger` (in `core`) is the only sanctioned logging entry point.
It intentionally does not accept or format raw camera/audio/biometric
payloads — see `CODING_STANDARDS.md` for the full rule. This is a structural
decision, not just a style guideline: later phases that add face recognition
or currency recognition must pass already-redacted metadata to this logger.

## What Phase 1 explicitly does NOT include

- No AI models, no camera pipeline, no glasses SDK integration.
- No real encrypted storage yet (arrives in Phase 21) — `AppSettings` is a
  plain data class with no persistence in Phase 1.
- No network/online code path (arrives in Phase 22).

## Known limitation of this build environment

This project was scaffolded in a sandbox without the Android SDK or network
access to Google's Maven repositories, so the Gradle build has **not** been
compiled or run here. Per the roadmap's division of responsibility, running
the actual build and confirming the app launches on a device/emulator is the
product owner's task for this phase (see `VERIFICATION.md`).
