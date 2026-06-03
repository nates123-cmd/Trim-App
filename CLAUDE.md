# Trim — Project Context

## What it is
A calorie + weight tracking PWA. Forked from Tide (the suite's mindful-drinking app) by stripping everything except the **Fuel** (calorie) and **Body** (weight) engines. Built for a single user (Nate's girlfriend) — per-user login, her data only.

**Live URL:** https://nates123-cmd.github.io/Trim-App/
**Repo:** `nates123-cmd/Trim-App`
**Local dev:** `python3 -m http.server 8080` → http://localhost:8080 (SW bypasses cache on localhost)

## Lineage
Forked from `tide-app` at Tide cache `tide-v57`. Inherits Tide's suite infra verbatim:
- 8-digit email-OTP login + per-user RLS (shared Supabase project `xsmnfcmtbpeaccnyinkr`)
- Single-flight refresh-token fix (rotating refresh token race) — DO NOT undo
- `callClaude` → JWT-gated `/functions/v1/claude` edge proxy (Anthropic key server-side, never on the client)

## Structure
Single-file app. `index.html` (HTML+CSS+JS), `sw.js` (cache `trim-vN`, bump on deploy), `manifest.json`, `icon.svg` → `icons/*.png` + root PNGs.

## Three surfaces (bottom nav: Fuel / Weight / Profile)
- **Fuel** (home, `renderFuelTab`) — kcal ring + P/C/F macro bars, meal input ("describe what you ate" → Claude estimates kcal+macros), recents, derived food Library, today's meals (tap to edit, swipe to delete / →yesterday). Food-only energy (no alcohol).
- **Weight** (`renderBodyScreen`) — current weight + delta vs range + goal, SVG trend chart (1M/3M/6M/1Y), measurements grid, progress photos (PIN-gated). "Log weight" modal.
- **Profile** (`renderSettings`) — sex/age/weight/height/activity + goal weight + pace (Slow −250 / Leaning −500 / Maintain). Calorie goal = Mifflin-St Jeor TDEE + chosen deficit. Sign out.

## Data (shared suite Supabase, per-user RLS — reused Tide tables, no new schema)
- `tide_intake_logs` (category=`food`, kcal+macros in `metadata` jsonb) — meals
- `tide_body_metrics` — weigh-ins + measurements
- profile in localStorage + `user_settings` cloud mirror (goal_weight_lb, weight_pace, etc.)

## What was stripped from Tide
Drinking sessions, Sip/Indulge/Stack/Train tabs, Oura + Patterns, morning reflection, quotes, water/caffeine/supplements, history, Pulse home, log-anything FAB. The functions still exist in the file (dead) but are unreachable — boot/router/nav only touch Fuel/Weight/Profile. A later pass can delete the dead code to shrink the file.

## Deploy
1. Edit `index.html` (+ `sw.js`/`manifest.json` if needed)
2. Bump `CACHE_NAME` in `sw.js`
3. `git add . && git commit && git push` → GitHub Pages deploys in ~1 min

## Pending / verify
- First sign-in by a brand-new email depends on the project's **Confirm-signup** email template carrying `{{ .Token }}` (else she gets a link, not a code). Believed fixed project-wide; verify on her first login.
- In-app smoke test (sign in → log meal via AI → log weight → see trend) not yet run by a human.
- Dead Tide code (drinking/Oura/etc.) still present in index.html — optional cleanup.
- Residual cosmetic string: `callClaude` throws "Sign in to use Tide AI".
