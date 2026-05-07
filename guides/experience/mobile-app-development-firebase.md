# Mobile App Development Guide (AI Games + Firebase)

Last updated: 2026-05-03.

## Purpose

Describe the current implementation model for the AxiomNode mobile client and its Firebase-backed identity flow.

## Scope

Use this guide for client-module implementation concerns: module shape, platform asymmetry, configuration, and authentication setup.

Use the capability dossiers for cross-repo mobile orchestration and player identity behavior.

## Current implementation shape

The current repository is a Kotlin Multiplatform application, not a React Native or Expo project.

Observed structure under `mobile-app/App`:

- `composeApp/`: shared Compose Multiplatform UI, domain, DI, config, and networking code.
- `androidApp/`: Android launcher module.
- `iosApp/`: iOS host application.
- `commonMain/` and `jvmNativeMain/`: shared platform support code.

Key implementation signals in the build:

- Kotlin Multiplatform + Compose Multiplatform
- Ktor client for networking
- Koin for dependency injection
- Room/SQLite for local persistence
- Firebase Auth on Android-facing integration paths

## Security note

`EDGE_API_TOKEN` is an infrastructure secret and must never be embedded in mobile clients.

Current repository posture:

- `mobile-app` no longer consumes or generates `EDGE_API_TOKEN` in its build-time config.
- `/v1/mobile/*` routes are expected to stay public at the gateway layer.
- Any privileged behavior must rely on user auth or server-side credentials, not client-embedded secrets.

Cross-repo route ownership, BFF orchestration, and player-state semantics are documented in the player capability dossiers rather than repeated here.

## Current stack

- Kotlin Multiplatform
- Compose Multiplatform
- Ktor client
- Koin
- Room/SQLite
- Firebase authentication integration

## Firebase authentication flow

1. Android signs in through Credential Manager plus Firebase Auth.
2. The app obtains a Firebase-backed token plus user metadata.
3. The client syncs player state through the mobile profile endpoints.

Current platform asymmetry matters:

- Android has the concrete Firebase-backed sign-in implementation.
- iOS and desktop still rely on `AUTH_MODE=dev` for usable local sign-in.

## Local implementation notes

- Environment values are generated into `mobile-app/App/composeApp/env/*.properties`.
- `API_BASE_URL` must target the public edge, not an internal service hostname.
- `GOOGLE_WEB_CLIENT_ID` is required for Android Google Sign-In.

## Configuration baseline

- `API_BASE_URL`
- `AUTH_MODE`
- `FIREBASE_API_KEY`
- `FIREBASE_AUTH_DOMAIN`
- `FIREBASE_PROJECT_ID`
- `FIREBASE_APP_ID`
- `GOOGLE_WEB_CLIENT_ID`

These values are generated into platform-specific environment files under `mobile-app/App/composeApp/env/` and then compiled into the app configuration.

Do not expose internal infrastructure secrets in mobile-distributed configuration.

## Related documents

- [Mobile gameplay orchestration](../capabilities/player/mobile-gameplay-orchestration.md)
- [Player identity and profile](../capabilities/player/player-identity-and-profile.md)
- Repository-local references: `mobile-app/App/README.md` and `mobile-app/App/FIREBASE_SETUP.md`