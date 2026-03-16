# Swipers Production Tracker — Repo Setup

## What This Is

A single-file production tracker for the Swipers game development team. It's a self-contained HTML app with:
- Interactive dependency graph (DAG) showing task relationships across sprints
- List view with filtering, search, status cycling, and inline dependency editing
- Slide-over edit panel for full task management (title, owner, sprint, priority, status, dependencies)
- New task creation with blocker/downstream assignment
- Firebase Realtime Database live sync (optional — works offline with localStorage too)
- JSON export/import for backups
- Activity feed showing real-time team changes
- Marketing lane running parallel to dev tasks with cross-lane dependency support

## Project Structure

Set up the repo like this:

```
swipers-tracker/
├── index.html          ← The tracker (rename from swipers_tracker_live.html)
├── README.md           ← Setup instructions for the team
├── .gitignore
└── CLAUDE.md           ← This file
```

## Setup Steps

1. **Create a new GitHub repo** called `swipers-tracker` (or whatever name you prefer).

2. **Copy the tracker file** into the repo root as `index.html`:
   ```bash
   cp swipers_tracker_live.html index.html
   ```

3. **Create a `.gitignore`**:
   ```
   .DS_Store
   node_modules/
   *.log
   ```

4. **Create a `README.md`** with the following content (adapt as needed):

   ```markdown
   # Swipers Production Tracker

   Live production tracker for the Swipers development team.

   ## Quick Start

   1. Open https://[your-username].github.io/swipers-tracker/
   2. First time: paste your Firebase config (see below) and enter your name
   3. After that, it auto-connects on every visit

   ## Firebase Setup (One-Time, by project lead)

   1. Go to [Firebase Console](https://console.firebase.google.com)
   2. Create Project → name it `swipers-tracker`
   3. Build → Realtime Database → Create Database → pick region → Start in Test Mode
   4. Project Settings (gear) → General → Your Apps → click `</>` web icon → Register app
   5. Copy the config JSON object
   6. Share the config with the team (pin it in Discord)

   **Important:** Before Test Mode expires (30 days), go to Realtime Database → Rules and set:
   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```

   ## Usage

   - **Graph tab**: Visual dependency chart. Click a task to highlight its chain. Double-click to edit.
   - **List tab**: Filterable task list. Click the circle to cycle status (Todo → WIP → Done → Blocked).
   - **+ New Task**: Create tasks from either view. Set blockers and downstream deps on creation.
   - **Export/Import**: JSON snapshots for backups or sharing.
   - **Reset**: Reload default task data (useful when updating the tracker with new sprint plans).

   ## Team

   | Role | Owner Color |
   |------|------------|
   | Dev 1 | Blue |
   | Dev 2 | Purple |
   | Tech Art | Cyan |
   | Art/3D | Gold |
   | Full Team | Green |
   | Marketing | Pink |
   ```

5. **Enable GitHub Pages**:
   - Go to repo Settings → Pages
   - Source: Deploy from a branch
   - Branch: `main`, folder: `/ (root)`
   - Save
   - Your tracker will be live at `https://[your-username].github.io/swipers-tracker/`

6. **Commit and push**:
   ```bash
   git init
   git add .
   git commit -m "Initial tracker setup"
   git branch -M main
   git remote add origin git@github.com:[your-username]/swipers-tracker.git
   git push -u origin main
   ```

## How Data Versioning Works

The tracker has a `DATA_VERSION` constant near the top of the `<script>` block in `index.html`. When you update the default task data and bump this version string, every team member's browser will automatically clear their stale localStorage cache and load the new defaults on their next visit.

To push updated task data to the team:
1. Edit `INITIAL_TASKS` and/or `INITIAL_DEPS` in `index.html`
2. Change `DATA_VERSION` to a new string (e.g., `'v4_new_sprint'`)
3. Commit and push
4. GitHub Pages auto-deploys — team sees new data on next page load

If the team is using Firebase, one person should also click "Reset" after the deploy to push the new defaults to Firebase so everyone syncs immediately.

## Key Technical Details

- **Single file**: Everything (HTML, CSS, JS, data) is in one `index.html`. No build step, no dependencies, no server.
- **Firebase SDK**: Loaded from CDN (`firebase-app-compat` + `firebase-database-compat` v10.12.0). No npm install needed.
- **localStorage fallback**: Works fully offline. Firebase is optional for live sync.
- **No sensitive data**: Firebase config is not secret — it's a client-side SDK config. Security comes from Firebase Rules, not config secrecy.

## Modifying the Tracker

The code is organized in clearly labeled sections inside the `<script>` tag:

- `INITIAL_TASKS` — Default task array (id, sprint, title, owner, priority, detail, status)
- `INITIAL_DEPS` — Default dependency array ([fromId, toId, isCritical])
- `SPRINTS` — Sprint metadata (num, label, dates, phase, title)
- `OWNER_LABELS` / `OWNER_COLORS` — Team member config
- `PHASE_COLORS` — Sprint phase color coding
- `STATUS_COLORS` / `STATUS_ICONS` — Task status visual config

To add a new owner type (e.g., a contract developer):
1. Add to `OWNER_LABELS`: `contract: 'Contract Dev'`
2. Add to `OWNER_COLORS`: `contract: '#ff6b6b'`
3. Add a filter button in the HTML list toolbar
4. Bump `DATA_VERSION`

To add a new sprint:
1. Add to `SPRINTS` array with correct `num`, `label`, `dates`, `phase`, `title`
2. Add tasks with matching `sprint` number
3. Add dependencies
4. Bump `DATA_VERSION`
