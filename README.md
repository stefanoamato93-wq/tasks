# Tasks

My own task list, shaped like GoodTask: lists (categories), due dates, repeating tasks. Add tasks by voice with Siri on iPhone; they appear live on the web app (Windows + iPhone).

**Rule: every edit to this app must be documented in this README in the same change.**

- Live: https://stefanoamato93-wq.github.io/tasks/
- Repo: `stefanoamato93-wq/tasks` (branch `main`, GitHub Pages)
- Files: `index.html` (whole app, Firebase config inline), icons + `manifest.webmanifest`.

## Data
Firebase Realtime Database, project `groceries-d6616` (same as gym/grocery/calendar), node `taskApp/tasks/{id}`:

| Field | Meaning |
|---|---|
| `title` | text |
| `done` | bool |
| `doneAt` | ms, when closed |
| `list` | list name; empty = "Inbox" |
| `due` | ISO string. `YYYY-MM-DD` = all day; `YYYY-MM-DDTHH:MM[...]` = with time. A time of exactly 00:00 counts as all day (that is how Shortcuts sends all-day reminders). |
| `repeat` | `{ freq: day/week/month/year, every: n }` or empty |
| `lastDone` | ms, last time a repeating task was ticked |
| `notes` | text |
| `source` | `web` / `siri` / `goodtask` |
| `created` | server timestamp; Siri/import save without it and the app stamps it on first sight |

**No sign-in.** Anyone with the URL can read/write, like the gym app. Do not store anything sensitive.
The node is open in the RTDB rules: `"taskApp": { ".read": true, ".write": true }`.

`taskApp/lists/{key}`: `{ name, color }` (key = list name with `. # $ [ ] /` replaced by `_`). Lists without an entry get a colour from a fixed palette by name hash.

**Sub-tasks** live inside `notes` in GoodTask's format, one per line: `- [ ] text` / `- [x] text`. Other note lines are kept as plain notes. `parseNotes` / `buildNotes` read and write this.

## Features
- Sidebar (☰ on phone): Oggi (due today or late), Calendario, Programmati (has a date), Tutti, then one entry per list with open count. Last view is remembered (`localStorage.tasksView`).
- **List colours:** tap a list's dot in the sidebar to pick its colour. Tasks show it as a left border and a chip (in Oggi/Programmati/Tutti).
- **Calendar:** month grid (Monday start), tasks per day in list colour (dots on phone). Future repeats of repeating tasks are shown too (tick disabled on those; tick the real one). Completed tasks show on the day they were done. Tap a day to see its tasks below; quick add in this view puts the task on the selected day.
- **Sub-tasks:** "☑ 2/5" button on a task opens them inline with ticks; the edit sheet has a sub-task list (add, rename, tick, remove). Ticking a repeating task resets its sub-tasks.
- Open tasks are grouped: In ritardo, Oggi, Domani, Prossimi 7 giorni, Più avanti, Senza data. Completed in a collapsible section.
- Quick add puts the task in the current list (or due today in "Oggi").
- Tap a task to edit title, list, date/time, repeat, notes, or delete. Backdrop tap with unsaved changes asks Salva / Continua a modificare / Scarta.
- Ticking a repeating task moves its due date forward by the interval (skipping past dates) instead of closing it, with a "Prossima: …" toast.

## Siri Shortcut "Aggiungi task"
1. Richiedi input ("Che task?").
2. URL action: `https://groceries-d6616-default-rtdb.europe-west1.firebasedatabase.app/taskApp/tasks.json` (paste into a separate URL action; the URL slot of Ottieni contenuti often refuses a paste).
3. Ottieni contenuti dell'URL: POST, JSON body `title` = Richiedi input, `done` = false, `source` = `siri`. Optional `list`, `due`.

## Import from GoodTask (Apple Reminders) - Shortcut "Importa GoodTask"
Trova promemoria (Non è completato) -> Ripeti con ognuno -> URL + Ottieni contenuti (POST JSON): `title` = Elemento ripetizione, `list` = Elemento ripetizione > Elenco, `due` = Elemento ripetizione > Data di scadenza (formato ISO 8601), `notes` = Elemento ripetizione > Note, `done` = false, `source` = `goodtask`.
Needs iPhone Settings > App > Comandi > Avanzate > "Consenti condivisione grandi quantità di dati". Run once (each run adds everything again).
Shortcuts cannot read a reminder's repeat rule, so repeats are set by hand in the app.

## Import notes (31 Oct 2026)
The import shortcut sent dates as Italian text with the title stuck on (`1 nov 2026, 00:00Affitto`). All 184 were rewritten once to `YYYY-MM-DD[THH:MM]`, and `dueDate` also reads that Italian format so a future import still works.

## Changelog
- v4 (31 Oct 2026): list colours, calendar view, sub-tasks from notes, reads Italian Shortcuts dates. Removed the Siri icon.
- v3 (31 Oct 2026): GoodTask structure: lists sidebar, due dates with groups, repeating tasks, notes, edit sheet.
- v2 (31 Oct 2026): removed sign-in. Moved from Firestore (`finance-stefano`) to Realtime Database (`groceries-d6616`, node `taskApp`). Siri Shortcut is now one request.
- v1 (31 Oct 2026): MVP. List, add, tick, delete, live sync, Siri via Shortcut (Google sign-in).
