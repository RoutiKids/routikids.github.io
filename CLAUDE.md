# rotina-infantil-na-tv

Single-page app (`index.html`) served via GitHub Pages from `main`. `config.json` is also written by the app itself (commits "Atualiza config.json (app Rotina)"), so always fetch/pull before pushing.

## Language conventions

- Code comments: **English**.
- Commit messages, PR titles/descriptions and GitHub comments: **English**.
- User-facing UI text inside the app stays in **Portuguese (pt-BR)**.

## Versioning

- On every change to `index.html`, update the "Versão do código: YYYY-MM-DD HH:MM" line (near the bottom of the settings screen) to the current date/time in Europe/Berlin (the owner lives in Germany). The owner uses it to confirm on the TV that the latest version is live.

## Cloud sync (pairing)

- Devices pair with a 6-digit code (or QR link `?parear=CODE`) and then share one "house" in a Firebase Realtime Database, via its REST API (no SDK). Live updates come over Server-Sent Events.
- Each device signs in anonymously through the Firebase Auth REST API (no accounts). Only devices listed in `houses/<id>/members` can read or write that house, so removing a member cuts its access for real. Config lives in `houses/<id>/data`.
- `CLOUD_DB_URL` and `FIREBASE_API_KEY` in `index.html` configure it (the API key is public by design); an empty URL disables the feature and the old GitHub flow is shown instead.
- Security rules live in `database.rules.json` and must be pasted into the Firebase console whenever they change. Anonymous auth must stay enabled, with automatic cleanup of anonymous accounts **off**.
