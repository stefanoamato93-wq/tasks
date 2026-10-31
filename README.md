# Tasks

My own task list, shaped like GoodTask: categories (lists), due dates, repeating tasks, sub-tasks. Dark mode only. Add tasks by voice with Siri on iPhone; they appear live on the web app (Windows + iPhone).

**Rule: every edit to this app must be documented in this README in the same change.**

- Live: https://stefanoamato93-wq.github.io/tasks/
- Repo: `stefanoamato93-wq/tasks` (branch `main`, GitHub Pages)
- Files: `index.html` (whole app, Firebase config inline), icons + `manifest.webmanifest`.

## Data
Firebase Realtime Database, project `groceries-d6616` (same as gym/grocery/calendar), node `taskApp`:

`taskApp/tasks/{id}`:

| Field | Meaning |
|---|---|
| `title` | text |
| `done` | bool |
| `doneAt` | ms, when closed |
| `list` | category name; empty = "Inbox" |
| `due` | `YYYY-MM-DD` = all day; `YYYY-MM-DDTHH:MM` = with time. A time of exactly 00:00 counts as all day. `dueDate` also reads the Italian Shortcuts text format (`1 nov 2026, 14:00`). |
| `repeat` | `{ freq: day/week/month/year, every: n }` or empty |
| `lastDone` | ms, last time a repeating task was ticked |
| `notes` | text; **sub-tasks live here** in GoodTask's format, one per line: `- [ ] text` / `- [x] text` (`parseNotes` / `buildNotes`) |
| `source` | `web` / `siri` / `goodtask` (not shown in the UI) |
| `created` | server timestamp |

`taskApp/lists/{key}`: `{ name, color, order }` (key = name with `. # $ [ ] /` replaced by `_`). Lists without an entry get a palette colour by name hash; lists without `order` sort after, by name.
`taskApp/settings`: `{ defaultList }` (falls back to `1 Urg And Imp`).

**New tasks without `created`** (Siri, anything saved outside the app) get stamped on first sight: `created`, plus `list` = default list and `due` = today when missing (not for `source: goodtask`). So a Siri task lands in the default list for the day the app first sees it.

**No sign-in.** Anyone with the URL can read/write, like the gym app. Do not store anything sensitive.
The node is open in the RTDB rules: `"taskApp": { ".read": true, ".write": true }`.

## Features
- **Views** (sidebar on computer, bottom bar on phone): Oggi (due today or late), Calendario, Programmati, Tutti, Completati, then the categories. Last view is remembered (`localStorage.tasksView`).
- **Oggi / Programmati / Tutti** are grouped by category (name + colour dot + count), in the user's order. **A single category** is grouped by day (In ritardo, then one header per day, then Senza data). Group headers collapse (state in `localStorage.tasksCollapsed`).
- **Search** box (🔍) searches title, notes and category across all tasks, open and completed, grouped by category.
- **Quick add** goes to the default category (★) due today; inside a category it goes to that category, due today; in the calendar it goes to the selected day. The hint under the box says where.
- **Task sheet** (tap a task; every change saves at once): title; date with Oggi / Domani / Nessuna and −/+1 giorno, settimana, mese, plus date/time pickers; category chips; repeat chips (Mai, ogni giorno/settimana/mese/anno) plus "Personalizza: ogni N unità"; sub-tasks (tick, rename, remove, add); notes; Duplica; Elimina.
- **Drag & drop** (computer): drag a task onto a category in the sidebar, onto a category header, or onto a calendar day.
- **Categories:** "✏️ Modifica elenchi" in the sidebar: ↑/↓ reorder, ✎ rename (updates every task in it), ★ set as default, tap the dot to change colour, + Nuovo.
- **Completed** tasks are hidden from every view except "Completati", which has "Elimina tutti".
- **Repeating tasks:** ticking moves the due date to the next occurrence (skipping past dates) and resets sub-tasks. The calendar also shows future repeats (tick disabled there).
- **Sub-tasks** show as "☑ 2/5" on the task; tap to open them inline with round ticks and a "+ sotto-task" field.

## Siri Shortcut "Aggiungi task"
1. Richiedi input ("Che task?").
2. URL action: `https://groceries-d6616-default-rtdb.europe-west1.firebasedatabase.app/taskApp/tasks.json` (use a separate URL action; the URL slot of Ottieni contenuti often refuses a paste).
3. Ottieni contenuti dell'URL: POST, JSON body `title` = Richiedi input, `done` = false, `source` = `siri`. Optional `list`, `due`.

## Import from GoodTask (Apple Reminders) - Shortcut "Importa GoodTask"
Trova promemoria (Non è completato) -> Ripeti con ognuno -> URL + Ottieni contenuti (POST JSON): `title` = Elemento ripetizione, `list` = Elemento ripetizione > Elenco, `due` = > Data di scadenza, `notes` = > Note, `done` = false, `source` = `goodtask`.
Needs iPhone Settings > App > Comandi > Avanzate > "Consenti condivisione grandi quantità di dati". Run once (each run adds everything again).
Shortcuts cannot read a reminder's repeat rule, so repeats are set by hand in the app.
31 Oct 2026: the first full import sent dates as Italian text with the title stuck on (`1 nov 2026, 00:00Affitto`); all 184 were rewritten once to `YYYY-MM-DD[THH:MM]`.

## Changelog
- v5 (31 Oct 2026): dark mode, GoodTask-like layout (category groups, round ticks, bottom bar on phone), task sheet with quick date/category/repeat/duplicate, drag & drop, rename/reorder/default categories, separate Completati view with "Elimina tutti", search, per-day groups inside a category, new tasks and Siri default to the ★ category for today. Removed the Siri setup panel.
- v4 (31 Oct 2026): list colours, calendar view, sub-tasks from notes, reads Italian Shortcuts dates.
- v3 (31 Oct 2026): lists sidebar, due dates with groups, repeating tasks, notes, edit sheet.
- v2 (31 Oct 2026): removed sign-in. Moved to Realtime Database (`groceries-d6616`, node `taskApp`).
- v1 (31 Oct 2026): MVP with Google sign-in (Firestore).
