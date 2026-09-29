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

- `initSchedule(cfg)` keeps today's progress by default: done tasks keep their real times and outcome, the current task keeps its start and extensions, and new tasks are fitted in (`mergeDayProgress`). It runs when a config arrives from another device, after a settings save and on load (from `rotinaDayState_v1`, saved each second by `saveDayState`). A new day passes `{ fresh: true }`.
- "Testar agora" is shared: it sets `testStartedAt` in the config (so it reaches every paired device, with no database rule change) and today's tasks run 1 minute each from that moment. Changes during a test keep it running; "Sair do teste", the test ending or 2 hours passing clear it, and the real day comes back with its saved progress (test progress is never saved). Backups never carry `testStartedAt`.
- Settings has "💾 Salvar cópia" / "📂 Restaurar cópia" (a JSON file with `kind: 'backup'` and the config); the welcome screen also offers the restore, for a wiped browser. A restore is pushed to the paired devices.
- Today's progress is shared too: a tap that changes it (Já terminei, Começar, the "Terminou?" answer, reordering, the deadline) calls `progressAction()`, which publishes the day snapshot to `houses/<id>/progress` (`state`, `seq`, `updatedAt`, `by`). Other devices apply it through `initSchedule(TASKS, { progress })` when its `seq` is higher (numbers, not clocks, because device clocks differ). Automatic steps (the dialog timeout, catching up) run on each device by itself. If the database refuses it (rules not updated), sharing stops quietly for that session and the pairing is untouched.

## Children

- The config has `children` (`{ id, name, icon, color, picturesOnly }`) and every routine has a `childId`. `normalizeConfig` turns older configs (a single `childName`, routines without `childId`) into one child `c1` that owns every routine; `childName` stays equal to the first child's name for older app versions. It runs on load, on configs received or restored, and on templates.
- One day per child: the schedule engine keeps the day it works on in globals (`tasks`, `pointer`, `awaitingStart`, ...) and draws into swappable element refs (`heroEl`, `timelineEl`, ...). Each child has a slot (`SLOTS`); `withChild(slot, fn)` swaps that child's day and elements in (`saveDay`/`loadDay`/`useView`), runs `fn` and swaps back. Between calls the globals hold the first child's day. A new per-day global must be added to `freshDay`, `saveDay` and `loadDay`. Config changes call `initAllChildren()`; the 1-second loop is `renderAll()`.
- With two or more children the main screen is hidden and `buildPanels()` makes one column per child in `#kids-grid` (sizes follow the column via `--w`); each column's buttons act on its child. The "Terminou?" dialog is shared: one child's question at a time (`confirmOwner`). The automatic reload waits until no child is mid-task, and a shared test ends when every child's test has ended. Confetti stays inside the child's column.
- Settings edit one child at a time (tabs, up to 4 children): name, picture, colour, "só figuras" (the list shows only pictures), "mostrar a rotina" (`hidden`), and only that child's routines and weekly table. A hidden child keeps its routines but gets no slot (no column, sounds or questions); at least one child is always shown.
- Each column is filled with the child's colour (coloured header, tinted cards) and has its own ◀ ▶ arrows when no task is running. "Já terminei" is a round ✔ next to the minutes, the minute blocks get a full-width row, and tasks done before the last one collapse into a row of small pictures (`.kid-trail`) so the list fits without scrolling (scrolling with a TV remote is unreliable).
- The page declares `color-scheme: light dark` (meta and `:root`) although it only has light styles: some TV browsers darken pages on their own, and those built on Android WebView skip pages that say they handle dark mode. Keep every background explicit so the page stays light. Routine overlap warnings only compare routines of the same child.
- Today's progress is kept per child: locally in `rotinaDayState_v2` (`{ childId: snapshot }`, migrated from `rotinaDayState_v1`) and in the database at `houses/<id>/progress/<childId>`. A device whose progress is ahead of the stored entry publishes it when it reads the house, so a device joining mid-day starts from where the others are.
- Per child, "Terminou?" with no answer follows `confirmDefault`: `yes` (count as done, the old behaviour), `more` (extra time once, then closes as late) or `wait` (no timer, until the routine's deadline, then closes as late); `confirmSeconds` (10/20/30/60) and `extraMin` (3/5/10, also used by the "Não" button).
- A cloud read that started before this device's last config push is ignored for the config (`lastConfigPushAt`), so it cannot undo a save that was just made.
- Feedback uses pictures, not words: extra time sends the child's `runner` (🏎️ 🚀 🐕 🐇, chosen per child) racing across their column with a synthesized "vroom"; a task finished late pops the late icon with a soft sound. There is no text banner or speech synthesis (it was hard to understand). In a column, "Já terminei" is a box to tick ("Terminei"), shown ticked for 0.6 s before the task closes.
