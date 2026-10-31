# Tasks

My own task list. Add tasks by voice with Siri on iPhone; they appear live on the web app (Windows + iPhone).

**Rule: every edit to this app must be documented in this README in the same change.**

- Live: https://stefanoamato93-wq.github.io/tasks/
- Repo: `stefanoamato93-wq/tasks` (branch `main`, GitHub Pages)
- Files: `index.html` (whole app, Firebase config inline), icons + `manifest.webmanifest`.

## Data
Firebase Realtime Database, project `groceries-d6616` (same as gym/grocery/calendar), node `taskApp/tasks/{id}`:
`{ title, done, source: "web" | "siri", created }`.
Siri saves without `created`; the web app stamps it the first time it sees the task.

**No sign-in.** Anyone with the URL can read/write, like the gym app. Do not store anything sensitive.

The node must be open in the rules (Firebase console > `groceries-d6616` > Realtime Database > Rules). Add this line next to the existing ones, keep the others, then Publish:

```json
"taskApp": { ".read": true, ".write": true }
```

## Siri Shortcut
Recipe is also shown in the app under "Siri setup":
1. Ask for input ("Che task?").
2. Get Contents of URL: POST `https://groceries-d6616-default-rtdb.europe-west1.firebasedatabase.app/taskApp/tasks.json`, JSON body `title` = input, `done` = false, `source` = `siri`.

## Changelog
- v2 (31 Oct 2026): removed sign-in. Moved from Firestore (`finance-stefano`) to Realtime Database (`groceries-d6616`, node `taskApp`). Siri Shortcut is now one request.
- v1 (31 Oct 2026): MVP. List, add, tick, delete, live sync, Siri via Shortcut (Google sign-in).
