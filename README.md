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
| `repeat` | `{ freq: day/week/month/year, every: n, days?: [0-6, 0 = Sunday] (week only), mdays?: [1-31] (month only) }` or empty. "Giorni lavorativi" = `{ freq: week, every: 1, days: [1,2,3,4,5] }`. `nextOcc` finds the next date; a month day past the month's end (31 in November) falls on the last day. Plain monthly repeats clamp too (31 Jan -> 28 Feb). |
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
- **Views** (sidebar on computer, bottom bar on phone): Oggi, Domani (due tomorrow + repeats falling tomorrow, grouped by category), Calendario, Tutti, Completati, then the categories. Category names in the sidebar wrap instead of being cut (sidebar 220-320 px wide). The task sheet keeps the title and **Fatto** in a sticky header at the top. Last view is remembered (`localStorage.tasksView`).
- **Day bar in Oggi:** ‹ › arrows (and swipe left/right on phone) move to another day; "Oggi" jumps back. Today shows due + late tasks; another day shows tasks due that day plus future repeats falling on it (tick disabled). Quick add uses the shown day.
- **Delete:** red ✕ on every task deletes at once; the toast offers "Annulla" for 3 s. Same for the sheet's Elimina and for "Elimina tutti" in Completati (that one still asks once, since it is bulk).
- **Oggi / Tutti** are grouped by category (name + colour dot + count), in the user's order. **A single category** is grouped by day (In ritardo, then one header per day, then Senza data). Group headers collapse (state in `localStorage.tasksCollapsed`).
- **Search** box (🔍) searches title, notes and category across all tasks, open and completed, grouped by category.
- **Quick add** goes to the default category (`settings.defaultList`, falls back to `1 Urg And Imp`; no UI to change it) due today; inside a category it goes to that category; in Oggi it uses the shown day; in the calendar the selected day. **Phone:** no box at the top, the round + opens it above the keyboard and it stays open for several tasks (closes on tap elsewhere). **Computer:** box stays at the top for speed; press **N** to jump into it.
- **Task sheet** (tap a task; every change saves at once): title; date with Oggi / Domani / Nessuna and −/+1 giorno, settimana, mese, plus date/time pickers; category chips; repeat chips (Mai, ogni giorno/settimana/mese/anno) plus "Personalizza: ogni N unità"; sub-tasks (tick, rename, remove, add); notes; Duplica; Elimina.
- **Drag & drop** (computer): drag a task onto a category in the sidebar, onto a category header, or onto a calendar day.
- **Categories:** "✏️ Modifica elenchi" in the sidebar: ↑/↓ reorder, ✎ rename (updates every task in it, and the default if it was the default), tap the dot to change colour (centred swatch picker + "Altro colore"; the native picker alone opened off-screen), + Nuovo.
- **Calendar** opens on the current week: a rolling 6-week grid from this Monday; ‹ › (or swipe) move 4 weeks; past days are dimmed.
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
- v43 (3 Nov 2026): adding a task (phone and computer) saves it and opens its sheet at once, to set date, category, repeat (`openWhenReady`). Bulk mode: ☑ in the header (Esc or Chiudi to leave); tap tasks to select, "Tutti" selects the current view; bar actions Fatto, Oggi, Domani, +1 g, +1 sett, Lunedì, Senza data, Categoria…, Ripeti…, Elimina. One write per action, with "Annulla" (`bulkUpdate`). Category picker: every category is a same-size tile (min 46 px, grid), so the non-matrix ones are easy to tap.
- v41 (2 Nov 2026): category picker (editor and Sposta, `buildCatPicker`) grouped as 💼 Lavoro (Startup Routines; the 4 urgent/important lists as a 2x2 matrix: 🔥 Fai ora, 📅 Pianifica, 👥 Delega, 💤 Dopo; Awaiting, End) and 🏠 Vita (Start of Day, Todo, House, End of Day); unmapped lists go in Altri.
- v19 (31 Oct 2026): move a task to another category from any view: phone swipe left shows **Sposta** next to Elimina; computer has a ⇄ button and right-click; drag now drops on the whole group (header or its tasks), not just the header. `openMove()` picker; moves offer Annulla.
- v18 (31 Oct 2026): removed all haptics (`haptic`, `hapticTrigger`): they did not work on Stefano's iPhone and the overlay caused the tap-anywhere bug. The tick animation stays.
- v17 (31 Oct 2026): BUG FIX: on iPhone any tap on a task ticked it. The haptic overlay label was absolute but its circle was not `position:relative` (detached element, no computed style), so the label covered the whole row. Now `.ring` is `position:relative; overflow:hidden` and `hapticTrigger` always pins the parent. Ticking now shows the fill with a small pop and a strike-through for ~0.4 s before the task moves. Chart: bars now scale inside a fixed bar area (labels outside), so 38 and 33 no longer look the same.
- v16 (31 Oct 2026): "📊 Carico per giorno" view (sidebar, below the categories): bar chart of open tasks per day for the next 21 days plus an "In ritardo" bar, bars stacked by category colour, repeats included; tap a bar to open that day. Tapping a calendar day now opens that day's page (`openDay`). Day bar: the "Oggi" button moved to the left and only fades in/out, so ‹ › stay put and can be tapped repeatedly.
- v15 (31 Oct 2026): future-repeat circles look like the others (were dimmed); tapping one shows a toast instead of ticking. Task sheet date buttons add "Lunedì" / "Venerdì" = next Monday / Friday after today, keeping the time.
- v14 (31 Oct 2026): repeat marker on task rows is a small grey "↻" text instead of the 🔁 emoji; future repeats just say "ripetizione futura".
- v13 (31 Oct 2026): iPhone haptics fixed. A programmatic click on a hidden switch no longer ticks; Safari only ticks on a real tap on an `<input switch>`. `hapticTrigger(el)` lays an invisible switch label over: the round tick, "Aggiungi", the + button and the red "Elimina". Limits: the keyboard's Invio key and the full swipe cannot tick on iPhone (not taps); Android still vibrates via `navigator.vibrate`.
- v12 (31 Oct 2026): phone: after adding, the keyboard closes; haptic tick on add and when a swipe crosses the full-delete point (60% of the row; letting go there deletes, with undo). `haptic()` uses `navigator.vibrate` (Android) or the iOS 18 hidden `<input switch>` trick (iPhone Safari). Day swipes no longer start on a task row in either direction (rows are for delete); the date bar is sticky at the top on phone so days can always be swiped there, or on empty space, or with ‹ ›.
- v11 (31 Oct 2026): phone delete is now swipe-left on a task to show a red "Elimina", tap it to confirm (still with undo). The one-tap ✕ stays on computer only. A row swipe does not change the day; swipe on empty space or headers for that. Row markup: `li > .rowInner (ring, body, ✕) + .swipeDel`.
- v10 (31 Oct 2026): on computer the lists sidebar is sticky (follows the scroll; scrolls on its own if taller than the window).
- v9 (31 Oct 2026): finer repeats: Giorni lavorativi, weekly on chosen weekdays, monthly on chosen days of the month (works with "ogni N"); month-end clamp fix.
- v8 (31 Oct 2026): Domani view, sidebar names wrap, Fatto moved to a sticky header in the task sheet.
- v7 (31 Oct 2026): removed the star / default-list UI and the hint under the add box; phone add only via the + button; N shortcut on computer; undo toast 3 s.
- v6 (31 Oct 2026): red ✕ delete with undo, colour picker on screen, calendar starts on the current week, removed Programmati, day arrows + swipe in Oggi, fixed the Completati "Elimina tutti" bar.
- v5 (31 Oct 2026): dark mode, GoodTask-like layout (category groups, round ticks, bottom bar on phone), task sheet with quick date/category/repeat/duplicate, drag & drop, rename/reorder/default categories, separate Completati view with "Elimina tutti", search, per-day groups inside a category, new tasks and Siri default to the ★ category for today. Removed the Siri setup panel.
- v4 (31 Oct 2026): list colours, calendar view, sub-tasks from notes, reads Italian Shortcuts dates.
- v3 (31 Oct 2026): lists sidebar, due dates with groups, repeating tasks, notes, edit sheet.
- v2 (31 Oct 2026): removed sign-in. Moved to Realtime Database (`groceries-d6616`, node `taskApp`).
- v1 (31 Oct 2026): MVP with Google sign-in (Firestore).
