# Routikids

Single-page app (`index.html`) served via GitHub Pages from `main` at **https://routikids.github.io/**. Families share the same URL; each device keeps its routine in `localStorage`, and paired devices sync through Firebase (below).

## Old address

The app used to live at `fabiofialho1.github.io/rotina-infantil-na-tv/` (repository `fabiofialho1/rotina-infantil-na-tv`), which now only hosts a redirect page. It forwards each device's `rotina*` localStorage keys here in a `#migrate=` URL fragment (and keeps `?parear=CODE`). The small script at the top of `<head>` imports that data when this device has none of its own and removes the fragment: keep it, and keep the `rotina*` key names.

## Language conventions

- Code comments: **English**.
- Commit messages, PR titles/descriptions and GitHub comments: **English**.
- User-facing UI text lives in the `I18N` dictionary in `index.html`, in **Portuguese (pt-BR), English and German**; always add a new text to all three and use `tr('key', {vars})` (static HTML: `data-i18n`, `data-i18n-html`, `data-i18n-placeholder`, `data-i18n-title`). The translation function is `tr`, not `t`, because `t` is used everywhere for tasks.
- The language is per device (picker on the welcome screen and in settings; else the device language for new devices; devices that already had a routine default to Portuguese).
- UI texts never use dashes (—, –, or " - ") as punctuation: use a comma, colon, period or parentheses instead. Ranges such as `07:00 – 07:30` or `seg–sex` are fine.
- The app is called **Routikids** (`APP_NAME`), with a translated tagline (`tagline` key).

## Versioning

- On every change to `index.html`, update the `<meta name="app-version" content="Versão do código: YYYY-MM-DD HH:MM">` tag in `<head>` (keep that exact text: older app versions search for it to detect updates; the settings footer shows it translated) to the current date/time in Europe/Berlin (the owner lives in Germany). The owner uses it to confirm on the TV that the latest version is live, and the app compares it with the published `index.html` to detect updates (banner + automatic reload at a safe moment), so it must always move forward.

## Cloud sync (pairing)

- Devices pair with a 6-digit code (or QR link `?parear=CODE`) and then share one "house" in a Firebase Realtime Database, via its REST API (no SDK). Live updates come over Server-Sent Events.
- Each device signs in anonymously through the Firebase Auth REST API (no accounts). Only devices listed in `houses/<id>/members` can read or write that house, so removing a member cuts its access for real. Config lives in `houses/<id>/data`.
- `CLOUD_DB_URL` and `FIREBASE_API_KEY` in `index.html` configure it (the API key is public by design); an empty URL hides the pairing section (the app then works on a single device).
- New devices (nothing saved, not paired) see a welcome screen with a test/privacy note, routine templates (`ROUTINE_TEMPLATES`) and a pairing-code field. On TVs the code field comes first (with a QR code and the address to open the app on the phone) and the templates stay behind "Sem celular? Configurar aqui na TV"; existing devices never do.
- Phone-first setup: after "Começar" on a non-TV device, the "📺 Agora leve para a TV" screen (`openTvSetup`) shows the address to type on the TV, a pairing code that renews itself while open, and per-brand steps (`renderBrandGuide`); it turns into "TV conectada" when `announceNewMembers` sees the TV join. It also opens from settings. The "📖 Guia rápido" (`openGuide`) explains the app and the same TV steps.
- Security rules live in `database.rules.json` and must be pasted into the Firebase console whenever they change. Anonymous auth must stay enabled, with automatic cleanup of anonymous accounts **off**.
- A device only drops its pairing after two database refusals at least 15s apart, and only discards its anonymous identity when Auth says it is gone (not on generic 400s). Key events go to a per-device "Histórico da conexão" (localStorage) shown in the sync section, to diagnose lost pairings.

## Today's progress and backup

- `initSchedule(cfg)` keeps today's progress by default: done tasks keep their real times and outcome, the current task keeps its start and extensions, and new tasks are fitted in (`mergeDayProgress`). It runs when a config arrives from another device, after a settings save and on load (from `rotinaDayState_v1`, saved each second by `saveDayState`). A new day and "Sair do teste" pass `{ fresh: true }`; test mode is never saved.
- Settings has "💾 Salvar cópia" / "📂 Restaurar cópia" (a JSON file with `kind: 'backup'` and the config); the welcome screen also offers the restore, for a wiped browser. A restore is pushed to the paired devices.
