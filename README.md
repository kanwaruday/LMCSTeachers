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
(leave & other approvals, self-scoped), **My Score** (SS 50% + CW/HW
Regularity 50%, your own numbers only — no peer ranking yet, deliberately
deferred per Uday 2026-09-20 to validate the numbers first; explicitly
labeled draft — see `LMCSManagement/apps-script/teacher-portal.gs`), **My
Evaluations** (own
Teacher SS scores, `action=myssstats`), **My CW/HW Patterns** (own tag
frequency, reads the existing public CWHW proxy client-side), **Upcoming
Events** (campus official Calendar, `action=myupcomingevents`), **Your
Timetable** (see `data/timetable.json` below). Not yet built: Period
Adjustments (no existing process to digitize — needs real design first), a
Forms Hub, and target-based CW/HW coaching (needs Uday to define expected
ranges per tag). See `LMCSManagement`'s vault note
(`work/active/lmcs-management.md`) for the full phased plan.

## Identity resolution — EmployeeCode, not name

As of 2026-09-20, every "which record is mine" match in this app resolves an
EmployeeCode first, rather than comparing name strings — per Uday, after this
session repeatedly hit name-matching gaps (`"Seema"` vs `"SEEMA MAAM"` for one
EmployeeCode was a real example in the timetable source). Two layers:

1. **"Who am I" (the caller's own code):** `action=myemployeecode` — checks
   `EmpPersonal`'s `AuthEmail` column against the caller's verified Google
   sign-in email first (97.7% populated, confirmed against a live export),
   falling back to name-matching only if that's blank for a given person.
   **Not** `EmpMaster`'s own `AuthEmail`-named column — that one is a red
   herring, every value in it turned out to be a sequential row number, not
   an email; reading from there would have silently degraded to
   name-matching for everyone with no error to reveal it.
2. **"Which record refers to someone else"** (an SS sheet row, a CW/HW log
   entry): still resolved by matching that row's typed name against the
   Employee Roster Proxy's canonical name list — this side can't avoid
   name-matching, since neither source system captures a code or email at
   all. `resolveCodeFromName_` here mirrors `apps-script/teacher-portal.gs`'s
   `tpResolveEmployeeCode_` — keep them in sync if the normalization changes.

## `data/timetable.json`

Static asset, not live data — a personal-timetable lookup keyed
`campusId -> TeacherEmployeeCode -> {periods:[...]}` (re-keyed 2026-09-20;
was name-keyed before), generated once from `all-campuses__timetable.csv`
(the `lms-timetable-extractor` skill's output). No LMS1 data yet — the source
extraction hasn't covered that campus. **Code-keying tradeoff:** 562 of the
original 3,144 timetable rows had an assigned teacher but no EmployeeCode the
extractor could confidently resolve — those periods are simply absent now
rather than shown under a guessed name; teacher coverage per campus dropped
accordingly (e.g. LMS4 went from 31 named teachers to 14 with a resolved
code). Regenerate by re-running the extractor on a fresh export and
re-running the conversion (campus `"LMS 2"` → `"LMS2"`, day names normalized
to `Mon`..`Sat`, `BREAK` rows and rows with no assigned teacher OR no
resolved `TeacherEmployeeCode` dropped).

## Deploy

Static site on GitHub Pages, same as every other LMS repo — push to `main`,
Pages serves it directly. No build step.
