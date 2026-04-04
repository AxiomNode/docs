# Mobile App Development Guide (AI Games + Firebase)

## Goal

Define how to build the AxiomNode mobile app to consume AI-generated games, authenticate users with Firebase, and track gameplay activity.

## Product scope

The app should support:

1. Firebase sign-up/sign-in.
2. Quiz and word-pass discovery.
3. AI-powered game generation flows.
4. Progress and gameplay stats tracking.

## Recommended architecture

Main path:

1. Mobile app -> `api-gateway`
2. `api-gateway` -> `bff-mobile`
3. `bff-mobile` -> `microservice-quizz` / `microservice-wordpass`
4. Identity/stats path -> `microservice-users`

Current edge routes:

- `GET /v1/mobile/games/quiz/random`
- `GET /v1/mobile/games/wordpass/random`
- `POST /v1/mobile/games/quiz/generate`
- `POST /v1/mobile/games/wordpass/generate`

## Security note

`EDGE_API_TOKEN` is an infrastructure secret and must never be embedded in mobile clients.

For production, use one of these patterns:

1. Public mobile edge with server-side Firebase JWT validation.
2. Public intermediary endpoint/BFF that injects internal credentials safely.

## Recommended stack

- React Native + Expo
- Strict TypeScript
- React Navigation
- TanStack Query
- React Hook Form + Zod
- secure local storage (`expo-secure-store`)

## Firebase authentication flow

1. User authenticates with Firebase provider.
2. App obtains Firebase ID token.
3. App sends token to backend session endpoint.

Related domain endpoints:

- `POST /users/firebase/session`
- `GET /users/me/profile`
- `GET /users/me/stats`
- `POST /users/me/games/events`

## Contract note

For generation requests, `categoryId` is a string (for example: `"23"`, `"27"`).

## Minimum MVP flow

1. Onboarding and Firebase login.
2. Home recommendations (random quiz/word-pass).
3. Quiz gameplay screen.
4. Word-pass gameplay screen.
5. Result submission + event registration.
6. Profile and progress summary.

## Configuration baseline

- `EXPO_PUBLIC_API_BASE_URL`
- `EXPO_PUBLIC_FIREBASE_API_KEY`
- `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN`
- `EXPO_PUBLIC_FIREBASE_PROJECT_ID`
- `EXPO_PUBLIC_FIREBASE_APP_ID`
- `EXPO_PUBLIC_APP_ENV`

Do not expose internal infrastructure secrets in public client variables.