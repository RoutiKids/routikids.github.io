# rotina-infantil-na-tv

Single-page app (`index.html`) served via GitHub Pages from `main`. Families share the same URL; each device keeps its routine in `localStorage`, and paired devices sync through Firebase (below).

## Language conventions

- Code comments: **English**.
- Commit messages, PR titles/descriptions and GitHub comments: **English**.
- User-facing UI text inside the app stays in **Portuguese (pt-BR)**.

## Versioning

- On every change to `index.html`, update the "Versão do código: YYYY-MM-DD HH:MM" line (near the bottom of the settings screen) to the current date/time in Europe/Berlin (the owner lives in Germany). The owner uses it to confirm on the TV that the latest version is live, and the app compares it with the published `index.html` to detect updates (banner + automatic reload at a safe moment), so it must always move forward.

## Cloud sync (pairing)

- Devices pair with a 6-digit code (or QR link `?parear=CODE`) and then share one "house" in a Firebase Realtime Database, via its REST API (no SDK). Live updates come over Server-Sent Events.
- Each device signs in anonymously through the Firebase Auth REST API (no accounts). Only devices listed in `houses/<id>/members` can read or write that house, so removing a member cuts its access for real. Config lives in `houses/<id>/data`.
- `CLOUD_DB_URL` and `FIREBASE_API_KEY` in `index.html` configure it (the API key is public by design); an empty URL hides the pairing section (the app then works on a single device).
- New devices (nothing saved, not paired) see a welcome screen with a test/privacy note and routine templates (`ROUTINE_TEMPLATES`); existing devices never do.
- Security rules live in `database.rules.json` and must be pasted into the Firebase console whenever they change. Anonymous auth must stay enabled, with automatic cleanup of anonymous accounts **off**.
- A device only drops its pairing after two database refusals at least 15s apart, and only discards its anonymous identity when Auth says it is gone (not on generic 400s). Key events go to a per-device "Histórico da conexão" (localStorage) shown in the sync section, to diagnose lost pairings.
