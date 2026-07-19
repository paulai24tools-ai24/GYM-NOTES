# Setbook — Client Training Ledger

**The trainer's notebook, digitized.**

A single-file, self-contained web app for personal trainers to manage client profiles, diet plans, workout programs, and progress tracking — with a matching read-only portal so clients can check their own plan. No backend, no signup, no dependencies to install: open the HTML file and it runs.

---

## 1. The Problem

Personal trainers running an independent practice typically track everything in a physical notebook: client details, diet preferences, workout splits, weight and measurement history. This works, but it doesn't scale, can't be searched, and lives in exactly one place — the trainer's bag.

Setbook is the smallest possible digital upgrade to that notebook: same content, same trainer-in-control workflow, but searchable, chartable, and swipeable on a phone mid-session.

## 2. Who It's For

| Role | What they need |
|---|---|
| **Trainer** | Fast client lookup, plan editing, and progress logging — usable one-handed, mid-session, on a phone, or at a desk on a laptop. |
| **Client** | A read-only view of their own diet plan, workout program, and progress chart, gated by a simple PIN. |

## 3. Core Features

**Trainer dashboard**
- Client list with search and archive/active filtering
- Per-client tabs: Profile, Diet Plan, Workout Program, Progress
- Diet plans tagged Vegetarian / Non-Vegetarian / Eggetarian
- Workout builder: add training days, each with a focus and an exercise table (sets × reps)
- Progress log: weight + chest/waist/hips/arms per entry, charted over time against the client's target weight
- **Quick Log**: a floating action button (mobile) to log a client's weight in two taps without opening their full profile
- Settings: change the trainer PIN, export/import a full JSON backup

**Client portal**
- PIN-gated login by name
- Read-only Overview (progress bar toward target), Diet, Workout, and Progress tabs

**Responsive by design, not by accident**
- One file, two layouts: a sidebar + detail dashboard on wider screens, a full-screen drill-down (list → client → back) on narrow ones
- Tabs are swipeable on touch devices (with a dot indicator) and clickable everywhere else — the same `SwipeTabs` component drives both the trainer and client views

## 4. Design System

The one thing this page is built to be remembered by: **diet-type tags reuse India's real FSSAI food-packaging symbol** — a small square outline with a colored center dot (green for vegetarian, brown for non-vegetarian) — instead of a generic colored label. It's a real-world mark people already recognize from every food package, repurposed as a functional UI tag.

| Token | Value | Use |
|---|---|---|
| `--ink` | `#15191B` | Page background |
| `--panel` / `--panel-2` | `#1E2523` / `#242B29` | Card and panel surfaces |
| `--line` | `#2C3432` | Hairline borders |
| `--chalk` | `#EDEFE9` | Primary text |
| `--muted` | `#8B9A94` | Secondary text |
| `--accent` | `#D9A441` | Primary actions, targets, highlights (a "stopwatch gold") |
| `--veg` / `--nonveg` / `--egg` | `#3FA34D` / `#A13D2C` / `#9C7A2E` | Diet marks + weight trend line |

**Type:** Barlow Condensed (display, uppercase, used sparingly for headings and brand) · Inter (body/UI) · IBM Plex Mono (all numeric data — weights, reps, dates — for tabular alignment and a "logbook" feel).

## 5. Data Model

Everything lives under one storage key as a single JSON object:

```json
{
  "trainerPin": "1234",
  "clients": {
    "<clientId>": {
      "name": "", "pin": "", "age": 0, "gender": "female",
      "heightCm": 0, "dietType": "veg", "goal": "",
      "startWeight": 0, "targetWeight": 0, "active": true, "notes": "",
      "dietPlan": { "breakfast": "", "lunch": "", "snacks": "", "dinner": "", "notes": "" },
      "workoutProgram": { "days": [
        { "day": "Monday", "focus": "Push", "exercises": [{ "name": "Bench Press", "sets": 4, "reps": "8" }] }
      ]}
    }
  },
  "progress": {
    "<clientId>": [
      { "date": "2026-07-19", "weight": 72.4, "chest": null, "waist": null, "hips": null, "arms": null, "notes": "" }
    ]
  }
}
```

## 6. Architecture

This repo's `setbook.html` is **fully self-contained**: React and ReactDOM are bundled directly into the file at build time (via esbuild — no CDN script tags, no build step required to run it). There are exactly two things loaded over the network, both optional and cosmetic: the Google Fonts stylesheet. Delete that `<link>` tag and the app still works, just in a system font.

- **No backend.** Data is persisted to the browser's `localStorage`, scoped per-device, per-browser.
- **No charting or icon library.** The progress chart and every icon are hand-rolled SVG — one less dependency to break or go stale.
- **Backup & transfer.** Because there's no shared server, Settings includes Export (downloads a `.json` snapshot of everything) and Import (restores from one). This is the manual bridge between a trainer's laptop and a client's phone until/unless a real backend is added.

## 7. Known Limitations

- **PINs are a UI gate, not security.** They're stored in plain text in local storage. Don't use this for anything beyond casual access control between a trainer and their own clients.
- **No cross-device sync.** A trainer's laptop and a client's phone each have their own local copy. Use Export/Import to move data between devices manually.
- **Single point of failure.** Clearing browser data (or a different browser/incognito session) loses everything not backed up via Export.

## 8. Roadmap (if this becomes more than a personal tool)

1. Real backend + auth (e.g. Supabase/Postgres) so trainer and client devices share live data — the data model above ports over directly.
2. Exercise library with autocomplete instead of free-text entry.
3. CSV export for progress history (spreadsheet-friendly, separate from the full JSON backup).

## 9. Running It

Open `setbook.html` directly in any browser — double-click it, or serve the repo with GitHub Pages and it works with zero configuration. No `npm install`, no build step, no server.

---

## Appendix: Original Build Prompt

The structured brief this project was built from — useful if you want to hand this to an AI assistant to extend, rebuild, or fork it:

> Build a client management web app for an independent personal trainer, replacing a paper notebook.
>
> **Roles:** Trainer (full edit access, PIN-gated) and Client (read-only view of their own data, PIN-gated by name).
>
> **Data per client:** name, PIN, age, gender, height, diet type (vegetarian / non-vegetarian / eggetarian), goal, start/target weight, free-text trainer notes, active/archived flag.
>
> **Diet plan:** free-text breakfast/lunch/snacks/dinner plus general notes, tagged with the client's diet type.
>
> **Workout program:** a list of training days, each with a focus and a table of exercises (name, sets, reps).
>
> **Progress tracking:** dated entries of weight plus optional chest/waist/hips/arms measurements, charted over time against the target weight, with full history editable/deletable.
>
> **Constraints:** must work equally well on a laptop (trainer's desk) and a phone (trainer's gym floor, client's pocket) — responsive layout, not just scaled-down; touch-swipeable tabs on mobile; a fast "quick log weight" shortcut for mid-session use.
>
> **Deployment constraint:** must run as a single downloadable HTML file with no build step, no backend, and no required external dependencies, so it can be committed to a GitHub repo and opened directly or served via GitHub Pages.
>
> **Design direction:** distinctive, not templated — avoid generic dashboard blue-and-white. Ground the visual identity in the real subject matter (a trainer's ledger) rather than generic SaaS design. Reuse a real-world visual convention if one fits (in this case: India's FSSAI green/brown vegetarian marking, repurposed as the diet-type tag).
