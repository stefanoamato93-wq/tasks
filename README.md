# Tasks

My own task list. Add tasks by voice with Siri on iPhone; they appear live on the web app (Windows + iPhone).

**Rule: every edit to this app must be documented in this README in the same change.**

- Live: https://stefanoamato93-wq.github.io/tasks/
- Repo: `stefanoamato93-wq/tasks` (branch `main`, GitHub Pages)
- Files: `index.html` (whole app), `firebase-config.js` (same Firebase project as Finance v2), icons + `manifest.webmanifest`.

## Data
Firestore, project `finance-stefano`, path `users/{uid}/tasks/{id}`:
`{ title, done, source: "web" | "siri", created }`.
Existing Finance v2 rules cover it (owner uid only). Siri saves without `created`; the web app stamps it the first time it sees the task.

## Sign-in
- Web app: Google sign-in (same as Finance v2).
- Siri: an email/password linked to the SAME Google account (one-time, from "Siri setup" in the app). Needs Email/Password turned on in Firebase console > Authentication > Sign-in method.

## Siri Shortcut
Recipe is shown in the app under "Siri setup" once linked:
1. Ask for input.
2. POST to Firebase Auth `signInWithPassword` -> `idToken`.
3. POST the task JSON to the Firestore REST URL with `Authorization: Bearer <idToken>`.

## Changelog
- v1 (31 Oct 2026): MVP. List, add, tick, delete, live sync, Siri via Shortcut.
