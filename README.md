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

**Phase 0 + 1a only** (My Requests — leave & other approvals, self-scoped to the
signed-in teacher's own submissions). See `LMCSManagement`'s vault note
(`work/active/lmcs-management.md`) for the full phased plan: Timetable, Period
Adjustments, Forms Hub, CW/HW tag-based coaching, live SS/incentive visibility,
and the gamification layer are later phases, not yet built.

## Deploy

Static site on GitHub Pages, same as every other LMS repo — push to `main`,
Pages serves it directly. No build step.
