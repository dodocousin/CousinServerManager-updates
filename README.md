# Cousin Server Manager

A Windows web interface for managing **ARK: Survival Ascended** dedicated
servers — start/stop, auto-updates, scheduled restarts, INI/launch parameter
editing, backups, mods, RCON, plugins, Discord notifications, player bans,
scheduled RCON commands, multi-user accounts, and more.

Everything runs locally on your machine and is controlled from a browser at
`http://localhost:<port>` (the port is chosen during installation).

---

## 📥 Download

**[⬇ Download the latest release](https://github.com/dodocousin/CousinServerManager-updates/releases/latest)**

Run `CousinServerManagerSetup.exe`, follow the setup wizard, then open the app
in your browser.

---

## 📑 Table of Contents

- [Dashboard](#-dashboard)
  - [Clusters](#clusters)
  - [Server Cards](#server-cards)
  - [Bulk Actions](#bulk-actions)
  - [Activity Log](#activity-log)
- [Server Detail Tabs](#-server-detail-tabs)
  - [Overview](#overview)
  - [Launch Parameters](#launch-parameters)
  - [INI Settings](#ini-settings)
  - [Auto-Update Monitor](#auto-update-monitor)
  - [Auto-Restart Scheduler](#auto-restart-scheduler)
  - [RCON Console](#rcon-console)
  - [Logs](#logs)
  - [Auto-Save & Backups](#auto-save--backups)
  - [Mods](#mods)
  - [API & Plugins](#api--plugins)
  - [Discord](#discord)
- [Copy Tools](#-copy-tools)
- [Stop Server Modal](#-stop-server-modal)
- [Global Toolbar Features](#-global-toolbar-features)
  - [Ban Manager](#-ban-manager)
  - [Scheduled Commands (RCON Scheduler)](#-scheduled-commands-rcon-scheduler)
- [Account Manager](#-account-manager)
- [Licensing](#-licensing)
- [Update / Announcement Feed](#-update--announcement-feed)
- [Support](#-support)

---

## 📊 Dashboard

The Dashboard is the home screen — it shows all your configured servers,
grouped into **clusters**, with live status, quick actions, and a running
activity log.

### Clusters

Clusters are **dashboard-only groupings** you create to organize your servers
(separate from the in-game ARK `-clusterID=` transfer setting, which lives in
Launch Parameters).

- Click **＋ Add Cluster** to create a new group and give it a name.
- Every server that isn't assigned to a cluster automatically appears under
  the **Unassigned Servers** bucket.
- Click a cluster tile to open it and see only that cluster's servers.
- Inside a cluster you can **✎ Rename** or **🗑 Delete** it (deleting a
  cluster never deletes the servers inside — they simply become unassigned).
- Reassign a server to a different cluster at any time using the
  **Dashboard Cluster** dropdown directly on its server card.

### Server Cards

Each server on the dashboard shows:

- **State indicator** (stopped / starting / running / stopping / updating)
  updated live over WebSocket
- Map name, player count, Game/RCON ports, Max Players, Cluster ID
- **Build ID** (currently installed Steam build)
- Live **RAM usage** and **uptime**
- Configured **CPU Affinity** and **CPU Priority**
- Quick action buttons: ▶ Start, ⏹ Stop, ↺ Restart, 💀 Force-Kill
- A checkbox to include/exclude the server from bulk actions
- **Details →** to open the full server detail view

### Bulk Actions

A toolbar lets you act on **all** servers or just the ones you've checked
(scoped automatically to the cluster you're currently viewing):

- **▶ Start All** — staggered start of every selected server
- **↺ Restart All** — saves world, stops, and restarts with a stagger between
  servers
- **⏹ Stop All** — opens the [Stop Server modal](#-stop-server-modal) scoped
  to the selected servers
- **⬆ Stop + Update + Start All** — stops each server, runs a SteamCMD
  update, then restarts it
- **🔐 Amazon Certificates** — downloads and installs the Amazon Root CA
  certificates + CRL required for ARK: Survival Ascended servers to
  authenticate correctly (run as Administrator for best results); shows a
  live install log
- **🗑 Delete Game Save** — permanently deletes world/tribe/profile save data
  for the selected (stopped) servers
- **🗑 Delete Cluster Folder** — permanently deletes the shared cluster
  transfer folder (uploaded characters & items waiting to transfer)

### Activity Log

A live-scrolling panel on the dashboard mirrors important background events
across **all** servers — state changes, auto-restart/auto-update triggers,
bulk operation progress — so you can monitor everything without opening each
server individually. Click **Clear** to reset it.

---

## 🖥 Server Detail Tabs

Opening a server ("Details →") reveals a tabbed view with the following
sections. Each of these tabs can be individually enabled/disabled per user
account in the [Account Manager](#-account-manager).

### Overview

- **Status** — current state, RCON online/offline, players online, local
  build ID
- **Configuration** — map, max players, game/query/RCON ports, cluster ID
- **Paths & Mods** — server directory, SteamCMD directory, active mod list
- **Install Tools** — download/install SteamCMD, install or reinstall the ARK
  server files (~30 GB) with a live install log
- **Process & CPU Performance** — view the live PID/process name, assign
  specific **CPU cores** (affinity mask) and a **CPU Priority**
  (Idle/BelowNormal/Normal/AboveNormal/High), apply immediately or save so it
  auto-applies on every future start
- **Live Resource Usage** — real-time RAM usage and uptime for the running
  process
- **Watchdog & Recovery** — automatic failure detection and recovery:
  - **Auto-Restart on Crash** — restarts the server automatically if it exits
    unexpectedly, with a configurable delay and a max-restarts-per-hour
    anti-loop limit
  - **Boot Timeout Watchdog** — force-kills (and optionally restarts) a
    server stuck in "starting" state for too long
  - **RCON Silence Restart** — force-restarts a "running" server if RCON
    becomes unreachable for a configurable number of minutes
  - **Resource Warning Thresholds** — logs a warning when CPU% or RAM exceeds
    limits you set
- **Uptime History** — a recorded log of start/stop events and how long the
  server stayed up each time

### Launch Parameters

A full visual editor for the ARK server command-line, organized into
sections, with a live preview of the generated command line:

- **Session Name** — the name shown in the server browser
- **Network & Connectivity** — `-MULTIHOME=`, `-PublicIPForEpic=`,
  `-crossplay`, `-UseServerNetSpeedCheck`
- **Gameplay** — `-ActiveEvent=` (holiday events), `-EnableIdlePlayerKick`,
  `-ServerPVE`, `-exclusivejoin` (whitelist mode), hide map player locations,
  `-ForceAllowCaveFlyers`, `-ForceRespawnDinos`, `-NoDinos`,
  `-AllowRaidDinoFeeding`, `-DisableAntiSpeedHack`
- **Data & Save** — `-AltSaveDirectoryName=`, `-usestore`, `-converttostore`
- **Logging & Admin** — `-servergamelog`, tribe log options,
  `-NotifyAdminCommandsInChat`, `UseDynamicConfig` + custom config URL
- **Performance & Misc** — `-NoHangDetection`, `-NoGameAnalytics`,
  `-AutoManagedMods`, `-NoBattlEye`, `-lowmemory`, under-mesh checking
  options, `-culture=`
- **Custom Arguments** — a free-text field for any extra flags not covered
  above, appended verbatim

Changes take effect the next time the server starts.

### INI Settings

A direct editor for `Game.ini` and `GameUserSettings.ini`:

- Collapsible per-file sections with add/edit/remove of individual keys, or
  add a brand-new `[SectionHeader]`
- **Search** across all settings with jump-to and next-result navigation
- If the server is **currently running**, saved changes are safely queued to
  a `.bak` file and automatically applied the next time the server stops
  (with a "Pending changes" banner so you don't lose track)
- **📁 Open Folder** to jump straight to the INI files in Windows Explorer
- **📋 Copy to...** — copy an entire file, whole sections, or individual
  key/value pairs to one or more other servers (see [Copy Tools](#-copy-tools))

### Auto-Update Monitor

Runs in the background (no browser tab required) and:

- Periodically checks Steam for a new build at a configurable interval
- Broadcasts a countdown warning message to players before restarting
- Sends a final shutdown broadcast, then updates and restarts the server
  automatically
- Lets you trigger a manual "check for update now"
- *(Requires an active license — see [Licensing](#-licensing))*

### Auto-Restart Scheduler

A background weekly restart scheduler per server:

- Enable/disable independently, runs even with the browser closed
- Configure a specific restart time for **each day of the week**
  individually
- **Countdown broadcast** warnings sent to players before the restart, with a
  customizable message template
- Optional **Destroy Wild Dinos** after the restart completes (sent via RCON
  once the server is back online)
- Optional **Update Server Before Restart** — runs a SteamCMD update as part
  of the restart sequence

### RCON Console

- A live interactive RCON console with command history (↑/↓ navigation)
- Quick-command buttons: `listplayers`, `SaveWorld`, `DoExit`, broadcast
  prompt, `GetGameLog`, `getchat`
- **Pre-made Commands** — pick from a library of common commands (kick, ban,
  unban, rename player, etc.), fill in the parameters through a form, preview
  the exact command, then send it to the current server or broadcast it to
  **all servers** at once
- **⚙ Manage Custom Commands** — build and save your own reusable command
  templates with `{placeholder}` parameters

### Logs

A live-streaming view of the server process's raw console output, with
autoscroll toggle and a clear button.

### Auto-Save & Backups

- **Auto-Save** — periodically sends `SaveWorld` via RCON at a configurable
  interval while the server is running
- **Auto-Backup** — runs continuously in the background:
  - Configurable interval, and independent retention limits for world/map
    backups vs. per-player profile backups (oldest are pruned automatically)
  - Optional `SaveWorld` before each backup
  - Custom backup storage path (with a folder browser)
  - **📸 Backup Now** for an on-demand manual backup
- **World Backup Management** — browse, refresh, open folder, **♻ Restore**,
  or **🗑 Delete** any world/map backup
- **Player Profile Management** — search backups by name or EOS ID, sorted
  newest-first; restore or delete individual player profile backups (player
  must be offline to restore)

### Mods

Two-panel mod manager:

- **CurseForge Mod Browser** (left) — search and browse ARK: Survival
  Ascended mods directly from CurseForge (requires a free CurseForge API
  key), with sorting and pagination
- **Server Mods** (right) — the active mod list for this server; add mods
  found in the browser, or add any mod manually by CurseForge ID; reorder,
  enable/disable, and save
- **📂 Folder** — open the server's mods folder in Explorer
- **📋 Copy to...** — copy the current mod list (replace or merge) to other
  servers

### API & Plugins

Integrates with **AsaApiLoader**, the third-party plugin framework for ARK:
Survival Ascended dedicated servers:

- **ASA Server API status** — shows whether `version.dll`,
  `AsaApiLoader.exe`, and `AsaApi.dll` are present, plus the latest available
  loader/API versions from GitHub
- **⬇ Download / Update API** — installs or updates the API files, with a
  live install log
- **Enable ASA API** toggle — marks the server to launch with the API
  loader injected (via `version.dll`) so all installed plugins load
  automatically on startup
- **Installed Plugins** grid — shows every plugin folder found in
  `ArkApi/Plugins`, whether its `.dll` and `config.json` are present:
  - **✏ Edit Config** — a smart auto-generated form (or raw JSON mode) to
    edit a plugin's `config.json` directly from the browser
  - **🔄 Reload** — sends the plugin's `{name}.reload` RCON command without
    restarting the whole server
  - **📋 Copy to...** — copy an entire plugin folder or just its
    `config.json` to other servers
- **📂 Open Folder** — jump to the Plugins directory in Explorer
- *(Plugin management requires an active license — see
  [Licensing](#-licensing). Need plugins? Use the 🧩 **Get Plugins** button in
  the top bar.)*

### Discord

Per-server Discord webhook notifications:

- Set a webhook URL (from your Discord channel's Integrations settings)
- Toggle and customize the message for each lifecycle event:
  ✅ Server Start, 🛑 Server Stopping, 🔎 Update Detected, ⬆️ Server Updating,
  💥 Server Crash / Unexpected Stop
- Message templates support placeholders like `{serverName}`, and
  `{localBuild}` / `{remoteBuild}` for update-detected messages
- **🔔 Send Test Message** to verify your webhook works

---

## 📋 Copy Tools

Several tabs include a "Copy to..." action for quickly applying settings
across your fleet of servers:

- **Copy Server Configuration** — pick individual categories/settings from
  the current server and push them to any number of other servers
- **Copy INI Settings** — copy an entire INI file, whole sections, or
  specific key/value pairs
- **Copy Mods** — replace or merge the mod list on other servers
- **Copy Plugin** — copy an entire plugin folder or just its config

---

## ⏹ Stop Server Modal

A confirmation modal used whenever you stop one or more servers:

- **Stop Immediately** — `SaveWorld` + `DoExit` right away
- **Warn Players First** — broadcasts a countdown message (via `serverchat`
  or `broadcast`) at a configurable interval before shutting down, with an
  option to remember your settings for next time

---

## 🌐 Global Toolbar Features

Available from the top bar on every page:

### 🚫 Ban Manager

A centralized player ban system:

- Search known players by name or EOS ID (auto-suggested from all servers'
  player-name history)
- Issue a ban with an optional reason, either **lifetime** or for a set
  number of hours
- **Target scope** — apply the ban to **all servers**, a specific
  **cluster**, or a single **server**
- Bans (and unbans) are sent live via RCON to every targeted server; if a
  server is offline, the ban is queued and automatically retried until it
  succeeds
- Expired timed bans are automatically lifted in the background
- **Ban History** view shows all past bans/unbans, and any active ban can be
  manually lifted at any time

### ⏰ Scheduled Commands (RCON Scheduler)

Schedule arbitrary RCON commands to run automatically:

- Create **recurring** tasks (run every N minutes) or **one-time** tasks (run
  once at a specific date/time)
- **Target scope** — send to all servers, a specific cluster, or a single
  server
- Enable/disable individual tasks, run any task immediately with ▶, and edit
  or delete tasks at any time
- Runs entirely in the background on a 60-second scheduler tick — no browser
  tab required

---

## 👤 Account Manager

Available at `/account.html` (via the **👤 Account** button):

- **My Account** — change your own password
- **License Status** *(master only)* — view license status, time remaining,
  machine hash, and activate or re-check a license key
- **Manage Users** *(master only)*:
  - Create additional user accounts with their own username/password
  - **Reset Password** or **delete** any non-master user
  - Per-user **allowed tabs** — toggle exactly which server-detail tabs
    (Overview, Launch Params, INI Settings, Auto-Update, Auto-Restart, RCON
    Console, Logs, Auto-Save & Backups, Mods, API & Plugins, Discord) that
    user can see
  - Per-user **allowed actions** — individually grant/revoke sensitive
    actions like Delete Game Save, Delete Cluster Folder, Edit Server,
    Delete Profile Backup, Copy Server Config, Rename Cluster, Delete
    Cluster
  - Per-user **allowed features** — grant/revoke access to the global
    **Ban Manager** and **Scheduled Commands** tools
  - The **master** account always has full, unrestricted access to
    everything

---

## 🔐 Licensing

A valid license unlocks certain advanced features (currently the
**Auto-Update Monitor** and **Installed Plugins management**). Activate or
check your license status any time from the [Account Manager](#-account-manager).
Locked features show a small banner explaining that a license is required.

---

## 🔔 Update / Announcement Feed

Every installed copy of Cousin Server Manager periodically checks a small
public [`updates.json`](./updates.json) feed hosted in this repo to know:

- Whether a newer app version is available (and where to download it)
- Any active announcements from the developer (new features, maintenance
  notices, important fixes, etc.), shown as small dismissible banners inside
  the app

This is a one-way, read-only feed — the app only ever reads this file, it
never writes back to it or to this repository in any way.

---

## 🛠 Support

For questions, plugins, or support, visit
[store.chezcousin.net](https://store.chezcousin.net).
