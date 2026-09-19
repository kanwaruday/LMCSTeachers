# LMCS Teachers

Teacher-facing self-service portal for La Montessori Schools — separate repo from
[LMCSManagement](https://github.com/kanwaruday/LMCSManagement) by design (distinct
ownership/branding, and a clean path to an app-store wrapper later), but **not** a
separate backend or auth system.

- **Auth:** loads `assets/auth.js` live from `LMCSManagement`'s GitHub Pages
  (`https://kanwaruday.github.io/LMCSManagement/assets/auth.js`) instead of a
  vendored copy, so there's exactly one session/allowlist implementation, not two
  drifting out of sync.
- **Backend:** calls the same "LMCS Principal's Daily Reporting Backend" Apps
  Script Web App every other module in `LMCSManagement` calls (see that repo's
  `apps-script/main.gs` / `apps-script/approvals.gs`). No new Apps Script project
  for this repo.
- **Access:** gated to the `Teacher` role specifically (see `init()` in
  `index.html`) — Principals/Coordinators/Owner keep using the full Approvals tab
  on `LMCSManagement`'s Principal's Daily Reporting page.

## Status

Built so far, all on the Home tab or its own bottom-nav tab: **My Requests**
(leave & other approvals, self-scoped), **My Evaluations** (own Teacher SS
scores, `action=myssstats`), **My CW/HW Patterns** (own tag frequency, reads
the existing public CWHW proxy client-side), **Upcoming Events** (campus
official Calendar, `action=myupcomingevents`, reuses `principal-dr.gs`'s
`pdrSchoolCalendarEvents_`), **Your Timetable** (see `data/timetable.json`
below). Not yet built: Period Adjustments (no existing process to digitize —
needs real design first), a Forms Hub, and target-based CW/HW coaching (needs
Uday to define expected ranges per tag). See `LMCSManagement`'s vault note
(`work/active/lmcs-management.md`) for the full phased plan.

## `data/timetable.json`

Static asset, not live data — a personal-timetable lookup keyed
`campusId -> teacherNameLower -> {periods:[...]}`, generated once from
`all-campuses__timetable.csv` (the `lms-timetable-extractor` skill's output).
No LMS1 data yet — the source extraction hasn't covered that campus.
Regenerate by re-running the extractor on a fresh timetable export and
re-running the conversion (campus `"LMS 2"` → `"LMS2"`, day names normalized
to `Mon`..`Sat`, `BREAK` rows and rows with no assigned teacher dropped,
teacher matched by lowercased name — same honest-not-found pattern the app
uses everywhere else if a name doesn't match).

## Deploy

Static site on GitHub Pages, same as every other LMS repo — push to `main`,
Pages serves it directly. No build step.
