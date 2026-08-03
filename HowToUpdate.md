# How to Publish an Update — Cousin Server Manager

This is your internal checklist for shipping a new version of Cousin Server
Manager to users who already have it installed. It covers version numbering,
building the installer, publishing a GitHub Release, and announcing the
update (or any general message) through the in-app notification banner.

This file is for **you** (the developer) — it lives in the private main repo
and is not shipped to end users.

---

## 1. Where version numbers live

There are **two separate version numbers** in this project. They are not
automatically linked — you must update both by hand.

| File | Key | Purpose |
|---|---|---|
| `package.json` | `"version"` | The version the **running app itself** reports. This is what gets compared against `latestVersion` in `updates.json` to decide whether to show the "update available" banner. |
| `installer.iss` | `#define AppVersion` | Purely cosmetic — shown in the installer wizard and in Windows "Apps & Features". Has **no effect** on the update-check logic. |

`build.ps1` does **not** set or prompt for a version — it just packages
whatever is currently sitting in those two files. So you must bump both
manually **before** running the build.

---

## 2. Full release checklist

1. **Bump the version** in `package.json`:
   ```json
   "version": "1.1.0"
   ```
2. **Bump the matching version** in `installer.iss` (keeps the installer UI
   in sync — good practice, not strictly required for the update-check to work):
   ```ini
   #define AppVersion   "1.1.0"
   ```
3. **Build the installer:**
   ```powershell
   .\build.ps1
   ```
   This produces `dist\CousinServerManagerSetup.exe`.
4. **Publish a GitHub Release** on the small **public** repo
   (`CousinServerManager-updates`) — NOT the private main repo:
   - Go to `https://github.com/dodocousin/CousinServerManager-updates/releases/new`
   - Tag: `v1.1.0` → "Create new tag on publish"
   - Title: `v1.1.0`
   - Notes: short changelog
   - Attach `dist\CousinServerManagerSetup.exe` (keep the exact same filename
     every time — the app's `downloadUrl` uses the `/releases/latest/download/`
     alias, which only works if the filename never changes)
   - Click **Publish release**
5. **Edit `updates.json`** in your local clone of `CousinServerManager-updates`:
   - Bump `"latestVersion"` to match step 1 (e.g. `"1.1.0"`)
   - Optionally add an entry to `"announcements"` describing what's new (see
     schema below)
   - `git add . && git commit -m "Bump to v1.1.0" && git push`

Once pushed, every installed copy of the app will pick up the new
`latestVersion` within a few hours (checked on startup + every 6 hours) and
show the "update available" banner with a link to your latest GitHub Release.

---

## 3. `updates.json` schema

This file lives in the **public** `CousinServerManager-updates` repo and is
fetched by every installed copy of the app via:
```
https://raw.githubusercontent.com/dodocousin/CousinServerManager-updates/main/updates.json
```

```json
{
  "latestVersion": "1.1.0",
  "downloadUrl": "https://github.com/dodocousin/CousinServerManager-updates/releases/latest/download/CousinServerManagerSetup.exe",
  "announcements": [
    {
      "id": "2026-08-01-new-feature",
      "date": "2026-08-01",
      "severity": "info",
      "message": "🎉 New: Discord notifications for server crashes! Check the Discord tab."
    }
  ]
}
```

| Field | Description |
|---|---|
| `latestVersion` | Compared against the running app's `package.json` version. If greater, triggers the "update available" banner. |
| `downloadUrl` | Where the "Download the update →" link points. Use the `/releases/latest/download/<filename>` alias so this URL never needs to change between releases. |
| `announcements` | An array — you can have zero, one, or many active at once. Each needs a **unique, permanent `id`** (never reuse an id, or already-dismissed banners for that id won't reappear for users who dismissed it before under that id). |

Each announcement object:

| Field | Description |
|---|---|
| `id` | Unique string. Recommended format: `YYYY-MM-DD-short-slug`. Once a user dismisses a banner, that `id` is remembered in their browser's `localStorage` and won't show again — so always use a **new** id for a new message. |
| `date` | Informational only (not used for any logic) — helps you keep track of when you posted it. |
| `severity` | One of `info`, `warning`, or `critical` — see section below. |
| `message` | The text shown in the banner. Supports basic HTML (e.g. `<strong>`, `<a href="...">`) since it's inserted directly into the page. |

---

## 4. Severity levels — what they mean and when to use them

The banner's color and icon change based on `severity`. Choose based on how
urgent/important the message is:

### 🔵 `info` (blue, 📢 icon)
General news, tips, or non-urgent updates. Nothing is broken, no action is
required from the user.

**Use for:**
- "New feature added: X"
- "Check out our Discord server for support"
- General announcements, promotions, or tips
- Routine "here's what's new" style messages

```json
{ "id": "2026-08-01-tip", "date": "2026-08-01", "severity": "info",
  "message": "💡 Tip: You can now schedule automatic restarts from the Auto-Restart tab!" }
```

### 🟠 `warning` (orange, ⚠️ icon)
Something the user should be aware of or plan for, but it's not breaking
anything right now. Also used automatically for the "update available" banner.

**Use for:**
- "Update available" (this one is generated automatically — you don't need
  to write this as an announcement, it's handled by `latestVersion`)
- Planned maintenance windows (e.g. license server downtime)
- Deprecation notices ("X setting will be removed in a future version")
- Recommended (but not mandatory) actions

```json
{ "id": "2026-08-05-maintenance", "date": "2026-08-05", "severity": "warning",
  "message": "⚠️ The license server will be down for maintenance Aug 10, 2-4 AM EST. Existing licenses will keep working normally during this window." }
```

### 🔴 `critical` (red, 🚨 icon)
Urgent — something is broken, insecure, or requires immediate action.

**Use for:**
- Security vulnerabilities that require updating immediately
- A bug that causes data loss, crashes, or corruption in the currently
  installed version
- "You must update now" type messages
- Critical service outages affecting core functionality (e.g. license
  validation completely down)

```json
{ "id": "2026-08-10-critical-bug", "date": "2026-08-10", "severity": "critical",
  "message": "🚨 Version 1.0.5 has a critical bug that can corrupt auto-backups. Please update to 1.0.6 immediately." }
```

**Rule of thumb:** if in doubt, use `info`. Reserve `critical` for things that
genuinely need the user's immediate attention — overusing it will make users
tune out the red banners over time.

---

## 5. Quick reference — full example `updates.json`

```json
{
  "latestVersion": "1.2.0",
  "downloadUrl": "https://github.com/dodocousin/CousinServerManager-updates/releases/latest/download/CousinServerManagerSetup.exe",
  "announcements": [
    {
      "id": "2026-09-01-v120-release",
      "date": "2026-09-01",
      "severity": "info",
      "message": "🎉 Version 1.2.0 is out! Adds Discord crash notifications and improved backup scheduling."
    },
    {
      "id": "2026-09-03-heads-up",
      "date": "2026-09-03",
      "severity": "warning",
      "message": "⚠️ Heads up: the default backup interval will change from 60 to 30 minutes starting in v1.3.0."
    }
  ]
}
```

---

## 6. Notes / gotchas

- The server checks for updates on startup (after a 5s delay) and then every
  **6 hours** for the lifetime of the running process — so changes to
  `updates.json` won't be picked up instantly by already-running installs,
  only within that window.
- The client polls the server's own `/api/updates/status` endpoint every
  **5 minutes** — this is just talking to your own server (cheap, local),
  not GitHub directly.
- Network failures when fetching `updates.json` are silent — the app keeps
  using the last successfully cached data and never crashes or blocks
  because of it.
- Never reuse an announcement `id` for a different message — always mint a
  new one (e.g. increment the date/slug) so it reaches everyone, including
  users who dismissed a previous announcement under an old id.
