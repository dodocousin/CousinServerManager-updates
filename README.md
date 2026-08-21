# Cousin Server Manager

**Current application version: 1.4.4**

Cousin Server Manager is a Windows-based web control panel for operating one
or many **ARK: Survival Ascended** dedicated servers. It combines server
lifecycle control, SteamCMD installation and updates, safe INI editing,
Beacon-backed advanced configuration, scheduled maintenance, backups, RCON
administration, mods, ASA API plugins, Discord alerts, player bans, multi-user
permissions, watchdog recovery, and UPS-aware emergency shutdown protection in
one local application.

The manager runs on the Windows host that runs your ARK servers and is
controlled from a browser at `http://localhost:<port>` (the port is selected
during installation). Background jobs continue to run when the browser is
closed.

> **Platform note:** Cousin Server Manager is designed for 64-bit Windows 10,
> Windows 11, and compatible Windows Server editions. Server process control,
> SteamCMD operations, certificate management, folder dialogs, and UPS
> protection use Windows and PowerShell facilities.

### Technical Architecture

- **Backend:** Node.js 18, Express, and authenticated WebSocket (`ws`) connections
- **Frontend:** plain HTML, CSS, and browser JavaScript with no frontend framework
- **Persistence:** JSON and JSONL files under the application `data/` directory
- **Windows integration:** PowerShell, SteamCMD, process controls, Registry,
  Explorer, certificate store, and Windows shutdown commands
- **RCON:** Source RCON through `rcon-srcds`
- **Beacon integration:** authenticated Beacon API access and Beacon v4 catalog
  data for advanced ARK configuration
- **Packaging:** `pkg` creates the standalone Windows executable and Inno Setup
  creates the installer
- WebSocket updates carry live state changes, logs, countdowns, installation
  progress, bulk-operation progress, and UPS telemetry

---

## 📥 Download

**[⬇ Download the latest release](https://github.com/dodocousin/CousinServerManager-updates/releases/latest)**

Run `CousinServerManagerSetup.exe`, follow the setup wizard, then open the app
in your browser.

---

## 📑 Table of Contents

- [First Run](#first-run)
- [Technical Architecture](#technical-architecture)
- [Dashboard](#dashboard)
  - [Server Definitions](#server-definitions)
  - [Clusters](#clusters)
  - [Server Cards](#server-cards)
  - [Bulk Actions](#bulk-actions)
  - [Activity Log](#activity-log)
- [Server Detail Tabs](#-server-detail-tabs)
  - [Overview](#overview)
  - [Launch Parameters](#launch-parameters)
  - [INI Settings](#ini-settings)
  - [Beacon Advanced Config](#beacon-advanced-config)
  - [Auto-Update Monitor](#auto-update-monitor)
  - [Auto-Restart Scheduler](#auto-restart-scheduler)
  - [RCON Console](#rcon-console)
  - [Logs](#logs)
  - [Auto-Save & Backups](#auto-save--backups)
  - [Mods](#mods)
  - [API & Plugins](#api--plugins)
  - [Discord](#discord)
- [Server Lifecycle & Reliability](#-server-lifecycle--reliability)
  - [Safe Operation Coordination](#safe-operation-coordination)
  - [Runtime Recovery](#runtime-recovery)
  - [SteamCMD Installation & Repair](#steamcmd-installation--repair)
- [Copy Tools](#-copy-tools)
- [Stop Server Modal](#-stop-server-modal)
- [Global Toolbar Features](#-global-toolbar-features)
  - [Ban Manager](#-ban-manager)
  - [Scheduled Commands (RCON Scheduler)](#-scheduled-commands-rcon-scheduler)
- [NUT / UPS Power Protection](#nut--ups-power-protection)
  - [Local NUT Runtime](#local-nut-runtime)
  - [Shutdown Policy](#shutdown-policy)
  - [In-Game & Discord Alerts](#in-game--discord-alerts)
  - [UPS History](#ups-history)
- [Account Manager](#-account-manager)
- [Licensing](#-licensing)
- [Update / Announcement Feed](#-update--announcement-feed)
- [Data, Updates & Security](#-data-updates--security)
- [Build from Source](#-build-from-source)
- [Suggested Roadmap](#-suggested-roadmap)
- [Troubleshooting](#-troubleshooting)
- [Support](#-support)

---

## First Run

1. Install and launch Cousin Server Manager.
2. Open `http://localhost:<port>` using the port selected in the installer.
3. Create the first **master administrator** account. Usernames require at
   least 3 characters and passwords require at least 6 characters.
4. Optionally activate a license during setup or later from **Account**.
5. Choose **Add Server**, enter the server/SteamCMD paths and unique game,
   query, and RCON ports, then save the definition.
6. Use the Overview install tools if SteamCMD or the ASA server files are not
   installed yet.
7. Review the launch parameters and INI files, then start the server.

The first account is permanently the master account. Additional restricted
accounts can be created later when the installation has a valid license.

---

## Dashboard

The Dashboard is the home screen — it shows all your configured servers,
grouped into **clusters**, with live status, quick actions, and a running
activity log.

### Server Definitions

Create, edit, or remove ASA server definitions directly from the browser. A
server definition can include the display/server name, map, server and SteamCMD
paths, game/query/RCON ports, RCON/admin credentials, maximum players, ARK
cluster ID and shared cluster directory, dashboard cluster assignment, CPU
affinity/priority, a GitHub raw INI synchronization URL, mods, and ASA API
settings. Native Windows folder/file pickers are available where a local path
must be selected.

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
- Active stop/restart countdown status when a warning sequence is running
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
- **Cancel Countdown** — cancel an active bulk stop/restart warning countdown
  before shutdown begins
- **⬆ Stop + Update + Start All** — stops each server, runs a SteamCMD
  update, then restarts it
- **🔐 Amazon Certificates** — downloads and installs the Amazon Root CA
  certificates + CRL required for ARK: Survival Ascended servers to
  authenticate correctly ; shows a
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

- **Status** — current state, RCON online/offline, players online, local and
  remote Steam build IDs
- **Configuration** — map, max players, game/query/RCON ports, cluster ID
- **Paths & Mods** — server directory, SteamCMD directory, active mod list
- **Install Tools** — download/install SteamCMD, install or reinstall the ARK
  server files (~30 GB), or verify/repair an existing installation with
  SteamCMD `validate`; all operations include a live install log, and buffered
  progress can be recovered if the browser reconnects
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
  - **Resource Warning Thresholds** — logs and optionally sends Discord
    warnings when CPU%, RAM usage, or free disk space crosses limits you set;
    low-disk alerts use a cooldown to avoid repeated notification spam
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
- **Data & Save / Cluster Transfer** — `-AltSaveDirectoryName=`, `-usestore`,
  `-converttostore`, and ARK cluster-transfer arguments
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

- Collapsible per-file sections with add/edit/remove of individual keys,
  including repeated keys, or add a brand-new `[SectionHeader]`
- **Search** across all settings with jump-to and next-result navigation
- **Raw Editor** for direct text editing, including large or heavily repeated
  settings that are easier to manage as plain INI text
- **Reload from Disk** when you want to discard the browser copy and re-read the
  current file
- If the server is **currently running**, saved changes are safely queued to
  a `.pending` file and automatically applied the next time the server stops
  (with a "Pending changes" banner so you don't lose track); pending changes
  can also be inspected or discarded before they are applied
- Before replacing an INI, the manager creates timestamped **Recovery
  Backups** that can be listed and restored from the browser
- Whole-file revision hashes protect against silently overwriting a file that
  was changed on disk after it was loaded in the browser
- **Repair RCON Settings** restores the required RCON values when needed
- **📁 Open Folder** to jump straight to the INI files in Windows Explorer
- **📋 Copy to...** — copy an entire file, whole sections, or individual
  key/value pairs to one or more other servers (see [Copy Tools](#-copy-tools))


### Beacon Advanced Config

Version **1.4.0** introduces direct **Beacon integration** for advanced
ARK: Survival Ascended configuration. The new Beacon Advanced Config interface
uses Beacon's catalog data while keeping the generated configuration inside the
normal Cousin Server Manager workflow.

- **Connect to Beacon** — authenticate the Server Manager with Beacon and keep
  the connection available for Beacon-backed configuration features
- **Beacon v4 catalog integration** — use Beacon's ARK configuration catalog as
  the reference for supported advanced settings and official/base values
- **Per-server state** — each configured ASA server/map keeps its own Beacon
  Advanced Config selections
- **Generated configuration preview** — review the resulting `Game.ini`,
  `GameUserSettings.ini`, and supporting launch-setting changes before they are
  applied
- **Safe INI application** — generated changes continue through the manager's
  normal revision checks, recovery backups, and pending-change workflow instead
  of bypassing the existing INI editor
- **Running-server protection** — when an affected INI cannot safely be replaced
  immediately, the changes are queued and applied through the normal `.pending`
  configuration system

#### Item Stat Limits

Beacon-backed **Item Stat Limits** can be configured directly from the Server
Manager. Individual item-stat indexes can be enabled and adjusted without
manually building every clamp entry.

For supported indexes, the interface can expose Beacon's base/official
**Raw Clamp** value as a reference while the administrator chooses the value to
apply.

The manager generates the appropriate item-stat clamp entries in `Game.ini`,
for example:

```ini
ItemStatClamps[0]=...
ItemStatClamps[1]=...
ItemStatClamps[2]=...
```

When at least one Item Stat Limit is enabled, the Server Manager also handles
the supporting settings needed for the clamp configuration to actually be used
by ASA:

```ini
?ClampItemStats=True
```

The required `ClampItemSets` launch setting is enabled alongside the selected
Item Stat Limits.

This avoids an incomplete setup where `ItemStatClamps[...]` values exist in the
INI but the server-side clamp feature was never activated.

### Auto-Update Monitor

Runs in the background (no browser tab required) and:

- Periodically checks Steam for a new build at a configurable interval
- Can check and apply an available update automatically before each start
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
- **⚙ Manage Custom Commands** — build, edit, and delete your own reusable
  command templates with `{placeholder}` parameters

### Logs

A live-streaming view of the server process's raw console output, with
autoscroll toggle and a clear button. Per-server log buffers are persisted so
recent output and operation progress remain available across browser reconnects.

### Auto-Save & Backups

- **Auto-Save** — periodically sends `SaveWorld` via RCON at a configurable
  interval while the server is running
- **Auto-Backup** — runs continuously in the background:
  - Configurable interval, and independent retention limits for world/map
    backups vs. per-player profile backups (oldest are pruned automatically)
  - Optional `SaveWorld` before each backup
  - Custom backup storage path (with a folder browser)
  - Temporary snapshot staging and retry handling for files ARK has briefly
    locked during backup creation
  - Automatic migration support for older backup directory layouts
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
  key), with a configurable ASA game ID, sorting, pagination, thumbnails, and
  mod metadata
- **Server Mods** (right) — the active mod list for this server; add mods
  found in the browser, look up/add any mod manually by numeric CurseForge ID,
  enrich manually entered IDs with CurseForge metadata, reorder, enable/disable,
  and save
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
- Optional **ASA API console** display for servers launched with the API
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

- Configure one main webhook, then optionally add **multiple destination
  webhooks per event**
- Enable, test, and customize each event independently
- Available events include server start/stopping/crash, update detected,
  update started/completed/failed, ban added/removed, resource warnings, low
  disk space, RCON online, watchdog recovery, scheduled restart starting, and
  NUT/UPS power events
- Message templates support contextual placeholders such as `{serverName}`,
  `{localBuild}`, `{remoteBuild}`, resource values, player/EOS data, and UPS
  telemetry (the UI lists the variables available for each event)
- Webhook delivery includes retry handling so a transient Discord failure does
  not interrupt the server operation that generated the notification

---

## 🛡 Server Lifecycle & Reliability

The manager does more than launch and terminate a process. It coordinates the
pre-launch, runtime, and shutdown steps needed to keep a fleet consistent.

- Creates missing default configuration and cluster directories
- Checks the game port before launch and verifies game/query/RCON ports during
  shutdown
- Optionally downloads a configured INI file from a GitHub raw URL before
  starting the server
- Applies pending INI changes before the next launch
- Optionally runs a Steam update before launch
- Builds and starts the ASA process without relying on generated batch files
- Polls RCON to distinguish a process that merely exists from a server that is
  ready for players
- Applies saved CPU affinity and process priority after startup
- Uses `SaveWorld` and `DoExit` for normal shutdown and can clean up a stuck
  process tree when graceful shutdown fails

### Safe Operation Coordination

Only one conflicting operation can control a given server at a time. Manual
actions receive an immediate busy response, while recurring background work is
placed into a per-server queue rather than being silently lost.

Automatic work is deduplicated and prioritized as follows:

1. Scheduled restart
2. Auto-update
3. Auto-backup
4. Auto-save

Different servers still process work independently, so a long task on one map
does not block unrelated maps.

### Runtime Recovery

The manager saves process IDs and runtime state to disk. If the manager is
restarted while ASA remains online, it attempts to adopt the existing process,
checks that it is still alive, and re-verifies it through RCON instead of
blindly displaying it as stopped or launching a duplicate instance.

### SteamCMD Installation & Repair

The Overview tab can:

- Download and install SteamCMD
- Install or update the ASA dedicated server application
- Verify and repair an existing installation with SteamCMD `validate`
- Prevent a verify/install operation while the server is active or another
  file operation owns that server
- Stream progress and buffered logs to the browser over WebSocket, including
  recovery of recent buffered progress after a page reconnect

The global **Amazon Certificates** tool installs and verifies the Amazon Root
CA certificates and CRL required by ASA authentication services.

---

## 📋 Copy Tools

Several tabs include a "Copy to..." action for quickly applying settings
across your fleet of servers:

- **Copy Server Configuration** — pick individual categories/settings from
  the current server and push them to any number of other servers
- **Copy INI Settings** — copy an entire INI file, whole sections, or
  specific key/value pairs; repeated key occurrences are preserved and the
  source revision is re-checked before the copy is applied
- **Copy Mods** — replace or merge the mod list on other servers
- **Copy Plugin** — copy an entire plugin folder or just its config

---

## ⏹ Stop Server Modal

A confirmation modal used whenever you stop one or more servers:

- **Stop Immediately** — `SaveWorld` + `DoExit` right away
- **Warn Players First** — broadcasts a countdown message (via `serverchat`
  or `broadcast`) at a configurable interval before shutting down, with an
  option to remember your settings for next time
- **Cancel Countdown** — abort an active warning countdown before the stop or
  restart reaches the shutdown phase

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

## NUT / UPS Power Protection

The global **NUT / UPS** page protects the entire Windows host from unsafe
power loss. Cousin Server Manager can install and own a local
[Network UPS Tools](https://networkupstools.org/) runtime, monitor the UPS,
warn players, save every running world, stop the ARK fleet, and optionally
shut down Windows when a configured emergency condition is reached.

> NUT/UPS management is a licensed, master-admin feature. NUT for Windows is
> still considered beta by its upstream project, and some USB devices require
> a compatible WinUSB/libusb driver binding before NUT can claim them.

### Local NUT Runtime

- Download and install NUT for Windows from the upstream project on demand
- Extract the runtime with the manager's bundled 7-Zip binary
- Start or stop the local NUT driver and `upsd` processes
- Auto-start the runtime whenever Cousin Server Manager starts
- Support USB/HID, SNMP/network, serial, or any installed custom NUT driver
- Scan for USB UPS devices and list available NUT drivers
- Choose automatic, native Windows HID, or libusb handling for `usbhid-ups`
- Supply additional sanitized `ups.conf` driver options when a device needs
  vendor/product IDs or another override
- Keep `upsd` local by default, with an optional LAN-listen setting
- Test the connection and open runtime/configuration folders from the UI

The live status panel reports available telemetry such as battery charge,
estimated runtime, UPS load, input/output voltage, battery voltage, NUT status,
on-battery duration, and the last successful poll.

### Shutdown Policy

Emergency conditions can use **ANY** enabled threshold or require **ALL**
enabled thresholds. Available checks include:

- Time continuously running on battery
- Remaining battery percentage
- Estimated battery runtime
- NUT's explicit `LOW BATTERY (LB)` status
- NUT/UPS communication loss for a configured duration, optionally only if
  the UPS was already known to be on battery

When the policy triggers, the manager:

1. Latches an emergency state and blocks new ARK starts/restarts.
2. Gives UPS protection priority over ordinary maintenance operations.
3. Sends the configured final player warning and waits the selected grace
   period.
4. Runs a graceful `SaveWorld`/`DoExit` shutdown for every active server and
   applies forced cleanup where required.
5. Optionally invokes Windows shutdown with configurable delay and forced-app
   closing behavior.

If Windows shutdown is disabled, all ARK servers remain stopped for an
administrator to inspect. Once an emergency shutdown sequence is latched,
restoring utility power does not unsafely restart or cancel work already in
progress.

### In-Game & Discord Alerts

Each UPS event can have its own in-game message and enable switch:

- Utility power lost/restored
- UPS communication lost/restored
- Low battery
- Battery percentage, runtime, or on-battery-duration shutdown threshold
- Communication-loss failsafe
- General/final emergency shutdown

Messages can use live variables such as battery percentage, runtime, UPS
name/status, and shutdown reason, and can be sent through `serverchat` or
`broadcast`. Power events are also written into the ARK server logs. The
Discord tab provides a general NUT event with optional per-event destination
webhooks.

### UPS History

Open **NUT / UPS → History** or `/nut-history.html` to view:

- 1-hour, 24-hour, 7-day, 30-day, and 90-day ranges
- Battery, estimated runtime, load, input/output voltage, and battery-voltage
  charts with tooltips
- Exact power, communication, low-battery, and shutdown event markers
- Configurable sampling interval and retention period
- Daily JSONL history files stored beneath the manager's `data/` directory
- Direct access to the history folder from the NUT / UPS interface

Large ranges are downsampled for responsive graphs while exact event records
are retained.

---

## 👤 Account Manager

Available at `/account.html` (via the **👤 Account** button):

- **My Account** — change your own password
- **License Status** *(master only)* — view license status, time remaining,
  machine hash, and activate or re-check a license key
- **Manage Users** *(master only)*:
  - Create additional user accounts with their own username/password
  - **Reset Password** or **delete** any non-master user
  - Manually **enable/disable** non-master accounts
  - Per-user **allowed tabs** — toggle exactly which server-detail tabs
    (Overview, Launch Params, INI Settings, Beacon Advanced Config, Auto-Update,
    Auto-Restart, RCON Console, Logs, Auto-Save & Backups, Mods, API & Plugins,
    Discord) that user can see
  - Per-user **allowed actions** — individually grant/revoke sensitive
    actions like Delete Game Save, Delete Cluster Folder, Edit Server,
    Delete Profile Backup, Copy Server Config, Rename Cluster, Delete
    Cluster
  - Per-user **allowed features** — grant/revoke access to the global
    **Ban Manager** and **Scheduled Commands** tools
  - Per-user **allowed clusters and servers** — grant individual maps or an
    entire dashboard cluster. A cluster grant automatically covers servers
    added to that cluster later
  - The **master** account always has full, unrestricted access to
    everything

---

## 🔐 Licensing

A valid machine-bound license unlocks advanced and commercial functionality,
including:

- Automatic Steam update monitoring
- Installed ASA plugin management
- Creation and management of additional user accounts and their permissions
- Global Ban Manager
- Scheduled RCON commands
- NUT / UPS power protection and historical monitoring

Core single-master server management remains available without a license.
Locked controls show a banner explaining that activation is required.

Activate or re-check a key from the first-run/login page or the
[Account Manager](#-account-manager). The account page displays the masked
key, machine hash, last check, expiry, and time remaining.

Licensing is machine-bound using a SHA-256 hash of the Windows Machine GUID
(with a host fallback) and supports lifetime and yearly product identifiers.
Activation/validation is performed against `licence-api.chezcousin.net`.

The manager checks a license immediately when required and then validates it
daily. After a successful validation, temporary license-service/network failures
receive a **72-hour / three-failure grace policy**; explicit expired, unknown,
invalid, or machine-mismatched responses are applied immediately. If a license
becomes invalid, non-master accounts are automatically disabled. Accounts
disabled specifically for that reason are restored when the license validates
again, while manually disabled accounts remain disabled.

---

## 🔔 Update / Announcement Feed

Every installed copy of Cousin Server Manager periodically checks a small
public `updates.json` feed hosted in the
[`CousinServerManager-updates`](https://github.com/dodocousin/CousinServerManager-updates)
repository to know:

- Whether a newer app version is available (and where to download it)
- Any active announcements from the developer (new features, maintenance
  notices, important fixes, etc.), shown as small dismissible banners inside
  the app

This is a one-way, read-only feed — the app only ever reads this file, it
never writes back to it or to this repository in any way.

The manager checks shortly after startup and every 6 hours. Browsers poll the
manager's cached result every 5 minutes, so every browser does not contact
GitHub independently. Network failures are non-fatal, and previously fetched
data remains available. Announcement banners support `info`, `warning`, and
`critical` levels and remember dismissed announcement IDs in browser
`localStorage`.

---

## 💾 Data, Updates & Security

### Persistent data

Runtime data is stored beside the installed executable in `data/`. It includes
server and cluster definitions, users, the session secret, license state,
runtime PIDs, app/CurseForge settings, Beacon connection/configuration state,
logs, uptime records, custom RCON commands, bans, schedules, backups (unless a
custom path is configured), and NUT configuration/history.

- Installer upgrades preserve the `data/` directory.
- Uninstall also preserves it by default so settings and saves are not
  silently destroyed.
- CurseForge settings are migrated from legacy root `config.json` storage to
  `data/app-config.json`, preventing an installer update from replacing the
  saved API key.
- Back up `data/` before moving the manager to another directory or machine.
  A machine-bound license may need to be transferred or reactivated after a
  hardware/Windows identity change.

### Authentication and access

- Passwords are stored as bcrypt hashes (cost 12), never as plain text.
- The browser uses an HTTP session cookie and a persistent random local
  session secret.
- All manager API routes except setup/login, license activation/status, and
  update-feed status require authentication.
- Server-scoped API routes enforce the user's server/cluster access in
  addition to hiding unavailable controls in the UI.
- The WebSocket connection reuses and validates the authenticated session.

### Network exposure

The default experience is intended for localhost or a trusted private LAN.
The built-in server uses HTTP rather than HTTPS. If remote access is required,
prefer a VPN or a properly configured HTTPS reverse proxy and firewall. Do not
port-forward the manager directly to the public internet without adding TLS,
access restrictions, rate limiting, and appropriate operational hardening.

Treat RCON passwords, admin passwords, CurseForge keys, Beacon authentication
data, Discord webhook URLs, license keys, plugin configs, and the complete
`data/` directory as sensitive. Never commit live credentials to source
control.

---

## 🧰 Build from Source

### Requirements

- 64-bit Windows
- Node.js 18 or newer
- npm
- PowerShell 7 or Windows PowerShell
- Inno Setup 6 for an installer build (the build script can download it)

### Development

```powershell
npm install
npm start
```

Then open `http://localhost:4000`. Set the `PORT` environment variable or
change `config.json` to use a different port.

### Tests

```powershell
npm test
```

The current automated suite validates automatic-task priority, deduplication,
failure isolation, per-server serialization, lock cleanup, and independent
execution across servers.

### Windows executable and installer

```powershell
.\build.ps1
```

The script installs npm dependencies, packages a Node 18 Windows x64 executable
with `pkg`, locates or installs Inno Setup 6, and creates:

```text
dist\CousinServerManagerSetup.exe
```

Before publishing, keep the version in `package.json` and `installer.iss` in
sync. See [`HowToUpdate.md`](./HowToUpdate.md) for the complete release
checklist and public announcement-feed format.

---

## 🗺 Suggested Roadmap

The following additions would build naturally on the existing architecture.
They are suggestions, not currently implemented features.

### Recommended next priorities

1. **Audit log** — persist the acting user, timestamp, source IP, target, and
   before/after configuration diff for sensitive actions, restores, deletes,
   account changes, and login failures.
2. **Readiness-aware rolling cluster maintenance** — update/restart one map,
   wait for RCON readiness and a healthy stabilization period, then continue
   to the next map; pause or abort if a server fails.
3. **Player-aware maintenance policies** — wait until empty, postpone while
   players are online up to a maximum deadline, and adapt warnings to player
   count.
4. **Backup verification and disaster-recovery bundles** — checksum/test ZIP
   files and export saves, INIs, server settings, mods, and plugin configs as
   one restorable package.
5. **Configuration profiles and revision history** — reusable PVE/PVP/event
   profiles, diffs, named versions, rollback, import/export, and apply-to-
   cluster actions.

### Additional ideas

- **Historical server metrics:** CPU, RAM, player count, RCON availability,
  disk/backup growth, crashes, and update duration; optionally expose a
  Prometheus endpoint for Grafana.
- **Disk forecasting:** estimate time until full, warn before a Steam update
  lacks space, and identify the largest save/mod/backup directories.
- **Central player administration:** current players across the fleet,
  kick/whitelist/message actions, first/last seen, session history, admin
  notes, and join/leave notifications.
- **Mod update intelligence:** detect CurseForge updates, show changelogs,
  report which servers use a mod, notify Discord, and coordinate safe restarts.
- **Plugin catalog:** approved plugin install/update/uninstall flows with
  compatibility checks and automatic config backup.
- **Maintenance calendar:** one timeline for restarts, updates, backups, and
  scheduled RCON commands, including timezone and conflict warnings.
- **Secure automation API:** scoped API tokens, generic outbound webhooks,
  Home Assistant integration, and a PowerShell/CLI client.
- **Role templates:** Viewer, Moderator, Operator, Backup Operator, Cluster
  Administrator, plus custom overrides.
- **Two-factor authentication:** optional TOTP for privileged accounts,
  recovery codes, active-session review, and session revocation.
- **Guided import/wizard:** scan an existing ASA installation, infer paths and
  ports, detect conflicts, suggest unused ports, and clone server definitions.
- **Manager self-update:** download a release, verify a published checksum or
  signature, back up `data/`, install, validate startup, and support rollback.
- **PWA/mobile notifications:** installable mobile UI with crash, update,
  backup, and UPS event notifications.

---

## 🧯 Troubleshooting

### The browser cannot open the manager

- Confirm `CousinServerManager.exe` is running.
- Check the console for the exact `http://localhost:<port>` address.
- Verify the selected port is not already used by another application.
- If accessing from another device, check the Windows Firewall and use the
  host's LAN IP instead of `localhost`.

### A server will not start

- Confirm `ArkAscendedServer.exe`, the server directory, and SteamCMD path are
  correct.
- Make sure each instance has unique game, query, and RCON ports.
- Review the server Logs tab and pre-launch messages.
- Use **Verify Server Files** while the server is stopped.
- Confirm the configured cluster and INI directories are writable.

### RCON stays offline

- Verify the RCON port and admin password match the launch configuration.
- Use **Repair RCON Settings** in the INI tab if required.
- Confirm Windows Firewall permits the RCON TCP port.
- Allow enough time for a large map/mod set to finish booting.

### CurseForge search fails or forgets its key

- Create/activate a key at the CurseForge developer console and accept its API
  terms.
- Save it from the Mods tab; current versions persist it in
  `data/app-config.json`.
- If a key was ever committed or shared, revoke and rotate it.


### Beacon connection or catalog loading fails

- Confirm the Server Manager host has working internet access to Beacon.
- Reconnect Beacon from the Beacon Advanced Config interface if the saved
  authorization is no longer valid.
- Reload the Beacon-backed configuration data if expected catalog values are
  missing.
- If Item Stat Limits were applied but do not take effect in-game, confirm the
  generated configuration includes `?ClampItemStats=True` and that
  `ClampItemSets` is enabled in the launch configuration.
- Review the Server Manager logs for Beacon authentication, API, catalog, or
  generated-configuration errors before manually changing the affected INIs.

### USB UPS detection fails

- Install the local NUT runtime first, then use **Auto-detect USB UPS**.
- Try the native Windows HID and libusb backends.
- Some devices require WinUSB/libusb binding through a tool such as Zadig;
  changing a USB driver can affect vendor software, so document the original
  driver and proceed carefully.
- Inspect the NUT driver and `upsd` logs from the runtime/state folders.
- For unsupported USB hardware, use the exact installed NUT driver and add
  vendor/product options shown by the scanner or NUT compatibility list.

### Automatic work appears delayed

Background jobs for one server are intentionally serialized. A scheduled
restart or auto-update takes priority over backup/save work, and automatic jobs
wait while a manual operation owns the server. Review the activity/server log
for `[TASK QUEUE]` messages rather than starting duplicate operations.

---

## 🛠 Support

For questions, plugins, or support, visit
[store.chezcousin.net](https://store.chezcousin.net).
