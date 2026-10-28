# Vision Aid Glasses

Offline-first wearable AI perception and assistance system for blind and
low-vision users. See `PRODUCT_SPEC.md` for the full product definition and
`Vision_Aid_Glasses_Master_Roadmap.md` for the phase-by-phase development
plan.

**Current state:** Phase 1 — Android Software Foundation. No AI models, no
glasses SDK, no cloud services are integrated yet — see `HANDOFF.md` for
exactly what exists right now.

## Repository Layout

```
vision-aid-glasses/
├── README.md                 (this file)
├── PRODUCT_SPEC.md            Phase 0 output
├── ARCHITECTURE.md            structural decisions, updated every phase
├── CODING_STANDARDS.md        conventions for all future phases
├── PROGRESS.md                phase-by-phase status tracker
├── HANDOFF.md                 latest phase summary — read this first
├── VERIFICATION.md            acceptance-criteria checklist per phase
├── .env.example                config/secrets template — copy, don't commit
├── android/VisionAid/          the Android Studio project
├── datasets/                   dataset provenance docs (Phase 7+)
├── hardware/                   hardware gap analysis (Phase 27+)
└── docs/verification/          per-phase verification detail, if split out
```

## Building

Requirements: Android Studio (current stable), JDK 17, Android SDK Platform
34, minSdk 26.

```bash
cd android/VisionAid
cp local.properties.example local.properties   # then set sdk.dir
./gradlew assembleDebug
```

> **Note:** this project was authored in a sandboxed environment without
> Android SDK access, so the build above has not actually been run yet. If
> `assembleDebug` fails, please report the exact error back so it can be
> fixed before Phase 1 is marked fully verified (see `VERIFICATION.md`).

## Running

Open `android/VisionAid` in Android Studio, select a device/emulator running
API 26+, and run the `app` configuration. On first launch the app requests
camera, microphone, and location permissions (all currently unused beyond
the permission-request proof-of-concept — real functionality starts at
Phase 3 onward) and displays a plain status screen confirming the foundation
booted successfully.

## Testing

```bash
cd android/VisionAid
./gradlew test          # unit tests across all modules
```

Each module (`core`, `ai`, `vision`, `audio`, `navigation`, `safety`,
`memory`, `documents`, `currency`, `glasses`) has its own `src/test` source
set. Phase 1 tests only cover the placeholder scaffolding and `core`'s data
classes — real coverage grows with each subsequent phase.

## Secrets / Configuration

Copy `.env.example` to `.env` (or populate `android/VisionAid/local.properties`
from its `.example`) before building anything that needs a real credential.
**Never commit populated `.env` or `local.properties` files** — `.gitignore`
already excludes them. No phase before Phase 22 actually requires a real
secret; the pattern exists now so no future phase is tempted to hard-code one.

## Continuing Development

This project follows a strict phase discipline — see
`Vision_Aid_Glasses_Master_Roadmap.md` Section 0.1. To resume work:

1. Read `PROGRESS.md` and `HANDOFF.md`.
2. Say "build next phase" (or open a new chat and upload
   `PROGRESS.md`, `HANDOFF.md`, `VERIFICATION.md`, `CODING_STANDARDS.md`,
   and `ARCHITECTURE.md` — the "latest builds pack").

Safety-critical phases (5, 16, 17, 18, 31) require your explicit sign-off in
`VERIFICATION.md` before being tagged complete — no exceptions.
