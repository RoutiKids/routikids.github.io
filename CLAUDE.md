# rotina-infantil-na-tv

Single-page app (`index.html`) served via GitHub Pages from `main`. `config.json` is also written by the app itself (commits "Atualiza config.json (app Rotina)"), so always fetch/pull before pushing.

## Language conventions

- Code comments: **English**.
- Commit messages, PR titles/descriptions and GitHub comments: **English**.
- User-facing UI text inside the app stays in **Portuguese (pt-BR)**.

## Versioning

- On every change to `index.html`, update the "Versão do código: YYYY-MM-DD HH:MM" line (near the bottom of the settings screen) to the current date/time in Europe/Berlin (the owner lives in Germany). The owner uses it to confirm on the TV that the latest version is live.

## Cloud sync (pairing)

- Devices pair with a 6-digit code (or QR link `?parear=CODE`) and then share one "house" record in a Firebase Realtime Database, via its REST API (no SDK, no accounts). Live updates come over Server-Sent Events.
- `CLOUD_DB_URL` in `index.html` points to the database; empty disables the feature and the old GitHub flow is shown instead.
- Security rules live in `database.rules.json` and must be pasted into the Firebase console whenever they change.
