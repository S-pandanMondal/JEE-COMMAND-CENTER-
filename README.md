# JEE Command Center

A single-file, offline-first study tracker for JEE (or any exam) prep — syllabus tracking, a daily planner, spaced revision, test score/percentile analytics, and optional group leaderboards to study alongside friends.

No build step, no backend to deploy, no dependencies to install. It's one HTML file that runs entirely in the browser.

## Features

- **Dashboard** — at-a-glance progress: daily study goal, streaks, weekly hours, and trend charts.
- **Syllabus** — organize chapters by subject, filter by status, and track completion.
- **Planner** — a to-do list for daily tasks, optionally tagged to a subject.
- **Revision** — spaced-repetition style tracking for lectures, with a per-lecture "comfort level" (A/B/C) so you know what needs another pass.
- **Tests** — log test scores per subject and overall percentile. Percentile trends are charted and automatically grouped by which combination of subjects each test covered, since percentile is only an apples-to-apples comparison within the same test pattern.
- **Community** — join one or more study groups by a shared group code and see a weekly leaderboard of hours studied. Fully optional; the app works completely fine with it untouched.
- **Study timer** — a running timer for study/revision sessions, tagged to a subject and (optionally) a specific chapter/lecture.
- **Themes** — Light, Dark, and AMOLED (true black, optimized for OLED screens). Cycle through them from Settings.
- **PWA-ready** — links a `manifest.json` so it can be "installed" to a home screen/desktop like a native app (see [Deployment](#deployment)).

## How it works

- Everything is a single `index.html` file: markup, CSS (custom properties for theming), and vanilla JavaScript — no framework, no bundler.
- **All personal data (syllabus, tests, logs, todos, settings) is stored in the browser's `localStorage`.** Nothing you track is sent anywhere unless you opt into a Community group.
- Rendering is a simple manual re-render pattern: app state lives in one `state` object, and `renderMain()` re-draws the current tab's HTML whenever state changes. There's no virtual DOM or component framework.
- Charts (hours trend, percentile progression) are generated as inline SVG, built directly from your state — no charting library.

## Community groups (optional, uses Supabase)

The Community tab is the one feature that talks to a server. It uses [Supabase](https://supabase.com) purely as a lightweight realtime datastore:

- Anonymous auth (no email/password) — a random member ID is generated per browser/install.
- Two tables: `groups` (just a group code) and `group_members` (display name, hours studied this week, total hours, keyed by group + member).
- Joining a group upserts your row; leaving deletes it. You can join multiple groups at once and switch between their leaderboards.

The Supabase project URL and anon key are already embedded in the file (anon keys are safe to expose client-side — that's what they're for). If you fork this project and want your own isolated group data, swap in your own Supabase project's URL/anon key and recreate the two tables above with Row Level Security policies that allow anonymous read/write scoped as you see fit.

If Supabase is unreachable or anonymous sign-in isn't enabled on the project, the rest of the app (syllabus, planner, tests, etc.) is unaffected — only the Community tab will show an error.

## Getting started

No installation required.

1. Download `index.html`.
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge).
3. Start adding subjects, chapters, and logging study time.

Your data persists in that browser via `localStorage`. To move your data to another device or browser, you'd currently need to manually export/import `localStorage` (there's no built-in export yet) — or just use the Community feature for the one piece of data (hours studied) that's meant to sync.

## Deployment

Because it's a static file, it can be hosted anywhere that serves static content:

- **GitHub Pages / Netlify / Vercel** — drop `index.html` (and `manifest.json` if you want the "install to home screen" prompt) into a repo/folder and deploy as static files.
- **Local use** — just double-click the file, or serve it with any static file server (`python3 -m http.server`, etc.).

To make it fully installable as a PWA (offline app icon, standalone window), you'd also want to add a service worker and icons referenced from `manifest.json` — the manifest link is already wired up, but a service worker isn't currently included.

## Browser support

Targets evergreen browsers (Chrome, Firefox, Safari, Edge — current versions). Relies on:
- CSS custom properties (theming)
- `localStorage`
- `fetch` (for the optional Supabase calls)
- ES2017+ JavaScript (async/await, arrow functions, template literals)

No transpilation, so very old browsers (IE11, etc.) are not supported.

## Project structure

Everything lives in one file for simplicity and easy portability:

```
index.html
├── <style>   — CSS custom properties (Light/Dark/AMOLED themes), component styles
├── <body>    — sidebar nav, topbar, settings panel, main content mount point
└── <script>  — app state, render functions per tab, event delegation, Supabase client, localStorage persistence
```

There's an intentional design tradeoff here: no build tooling means zero setup friction and the whole app can be read top-to-bottom, but it also means the file grows as features are added. If it keeps growing, splitting into modules with a lightweight bundler would be the natural next step — but that hasn't been necessary yet.

## Contributing / making changes

Since there's no build step, editing is direct: open `index.html`, find the relevant section (state, a `*Template()` render function, or the CSS theme block), and edit in place. Search for `function <tabName>Template()` to find where each tab's HTML is generated, and check the `switch(action)` block near the bottom for how buttons/clicks are wired up.

