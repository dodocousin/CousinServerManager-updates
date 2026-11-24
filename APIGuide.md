# Cousin Server Manager — API Guide

> **Status: preview.** API version `1`. Endpoints and webhooks listed here are
> available on builds that include the API.

The Cousin Server Manager API lets you build your own tools — Discord bots,
status pages, automation scripts, in-game integrations — that read and
control your ARK: Survival Ascended servers over HTTP.

---

## 📑 Table of Contents

- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Enabling API Access](#enabling-api-access)
- [API Keys](#api-keys)
  - [Creating a key](#creating-a-key)
  - [Scopes](#scopes)
  - [Server restrictions](#server-restrictions)
  - [Revoking and expiry](#revoking-and-expiry)
- [Making Requests](#making-requests)
  - [Base URL](#base-url)
  - [Authentication](#authentication)
  - [Response format](#response-format)
  - [Rate limits](#rate-limits)
- [Endpoints](#endpoints)
  - [Who am I](#get-me)
  - [List servers](#get-servers)
  - [Server details](#get-serversid)
  - [Server status](#get-serversidstatus)
  - [Online players](#get-serversidplayers)
  - [Activity logs](#get-serversidlogs)
  - [Start](#post-serversidstart)
  - [Stop](#post-serversidstop)
  - [Restart](#post-serversidrestart)
  - [Cancel countdown](#post-serversidcancel-countdown)
  - [Update a server](#post-serversidupdate)
  - [Update all servers](#post-serversupdate-all)
  - [Update progress](#get-maintenance)
  - [Backup](#post-serversidbackup)
  - [List world backups](#get-serversidbackupsworlds)
  - [List player profile backups](#get-serversidbackupsplayers)
  - [Restore a world backup](#post-serversidbackupsworldsrestore)
  - [Restore a player profile](#post-serversidplayerseosidrestore)
  - [RCON command](#post-serversidrcon)
  - [Broadcast message](#post-serversidbroadcast)
- [Webhooks](#webhooks)
  - [Setting up a webhook](#setting-up-a-webhook)
  - [Events](#events)
  - [Delivery format](#delivery-format)
  - [Verifying the signature](#verifying-the-signature)
  - [Retries, failures and auto-disable](#retries-failures-and-auto-disable)
  - [Who receives what](#who-receives-what)
  - [Managing webhooks with the API](#managing-webhooks-with-the-api)
  - [Minimal receivers](#minimal-receivers)
- [Error Codes](#error-codes)
- [Code Examples](#code-examples)
  - [PowerShell](#powershell)
  - [curl](#curl)
  - [JavaScript (Node.js 18+)](#javascript-nodejs-18)
  - [Python](#python)
- [Remote Access (Internet)](#remote-access-internet)
  - [Network & Remote Access settings](#network--remote-access-settings)
  - [Option A — Cloudflare Tunnel (no port forwarding)](#option-a--cloudflare-tunnel-no-port-forwarding)
  - [Option B — Caddy reverse proxy (automatic HTTPS)](#option-b--caddy-reverse-proxy-automatic-https)
  - [Built-in protections](#built-in-protections)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting](#troubleshooting)

---

## Requirements

- A **valid Cousin Server Manager license**. The API is a licensed feature;
  if the license lapses, all API keys stop working until it is reactivated.
- An account with the **API Keys** feature enabled (see below).
- Network access from your tool to the manager (same PC, your LAN, or the
  internet — see [Security Best Practices](#security-best-practices) before
  exposing it to the internet).

---

## Quick Start

1. Open the manager → **👤 Account** → **API Keys**.
2. Enter a name, pick the scopes you need, click **＋ Create API Key**.
3. Copy the key — it is shown **only once**.
4. Test it:

```powershell
$h = @{ Authorization = "Bearer csm_your_key_here" }
(Invoke-RestMethod "http://localhost:4000/api/v1/servers" -Headers $h).data
```

Replace `4000` with the port you chose during installation.

---

## Enabling API Access

| Account | How to get access |
|---|---|
| **Master administrator** | Always has access. |
| **Other users** | The master must enable the **API Keys** feature for that user in **Account → Manage Users**. It is **off by default**, even for users who have every other feature. |

A key can never do more than the account that owns it. If the master later
removes a permission from a user (a server, a tab, or the API Keys feature
itself), that user's keys lose it immediately. Deleting a user revokes all
of their keys.

---

## API Keys

### Creating a key

In **Account → API Keys**:

| Field | Description |
|---|---|
| **Name** | A label so you remember what uses the key (e.g. "Discord bot"). Max 60 characters. |
| **Expiry** | Never, 30 days, 90 days (default) or 1 year. |
| **Scopes** | What the key is allowed to do — see below. |
| **Servers** | All servers you can access (including servers added later), or a specific list. |

Keys look like this:

```
csm_1a2b3c4d5e6f_Xy9ExampleSecretDoNotUseXy9ExampleSecret12
└┬┘ └────┬─────┘ └──────────────────┬──────────────────────┘
prefix  key id                     secret
```

The manager stores only a fingerprint of the secret — **a lost key cannot be
recovered**; revoke it and create a new one. The `csm_<key id>` part is shown
in the key list so you can tell keys apart.

Each account can have up to **20 active keys**.

### Scopes

| Scope | Allows | Also requires the owner to have |
|---|---|---|
| `read` | List servers, details, status, online players | — |
| `logs` | Read server activity logs | **Logs** tab |
| `control` | Start, stop, restart, update, cancel countdowns | **Overview** tab |
| `rcon` | Send RCON commands and in-game messages | **RCON Console** tab |
| `backup` | Trigger backups, list backups | **Auto-Save & Backups** tab |
| `restore` | Restore world and player profile backups, list backups | **Auto-Save & Backups** tab |
| `webhooks` | Create and manage webhooks owned by this key | — |

Scopes your account doesn't have are greyed out when creating a key.

> ⚠️ The `rcon` scope can run **any admin command** on your server, and the
> `restore` scope **overwrites save data**. Only give them to tools you fully
> trust.

### Server restrictions

A key only sees servers that are allowed by **both** the key's server list
and the owner's own server permissions. Servers a key cannot see behave as
if they do not exist (`404 server_not_found`).

### Revoking and expiry

- Click **Revoke** next to a key — it stops working immediately.
- Expired and revoked keys stay visible in the list for 30 days, then are
  removed automatically.
- The list shows when and from which IP each key was last used.
- The master administrator sees and can revoke every user's keys.

---

## Making Requests

### Base URL

```
http://<manager-host>:<port>/api/v1
```

Examples: `http://localhost:4000/api/v1`, `http://192.168.1.50:4000/api/v1`.

### Authentication

Send your key in the `Authorization` header on every request:

```
Authorization: Bearer csm_your_key_here
```

Never put the key in the URL.

### Response format

Every response is JSON with the same shape.

**Success**

```json
{ "ok": true, "data": { } }
```

**Error**

```json
{ "ok": false, "error": { "code": "invalid_state", "message": "Server is stopped." } }
```

Always check `ok`, then use `error.code` (stable, for your code) or
`error.message` (human-readable, may change). See [Error Codes](#error-codes).

HTTP status codes:

| Status | Meaning |
|---|---|
| `200` | Success |
| `202` | Accepted — the action was started and runs in the background (start, stop, restart, backup) |
| `400` | Invalid request body or query |
| `401` | Missing, invalid, revoked or expired key |
| `403` | Key or account not allowed (scope, license, feature) |
| `404` | Unknown route, or server not found / not visible to this key |
| `409` | Action not possible right now (wrong state, another operation running) |
| `413` | Request body too large |
| `429` | Rate limited |
| `502` | The game server did not answer (RCON) |
| `500` | Unexpected manager error |

### Rate limits

| Limit | Value |
|---|---|
| Requests per key | **120 per minute** |
| Invalid key attempts per IP | **20 per 15 minutes**, then that IP is blocked (even valid keys) until the window ends |

Every authenticated response includes:

| Header | Meaning |
|---|---|
| `X-RateLimit-Limit` | Requests allowed per minute |
| `X-RateLimit-Remaining` | Requests left in the current minute |
| `X-RateLimit-Reset` | When the window resets (Unix time, seconds) |
| `Retry-After` | On `429` only — seconds to wait |

Tip: server status is refreshed by the manager every ~15–20 seconds, so
polling more often than that gains nothing.

---

## Endpoints

`{id}` is a server id, as returned by [`GET /servers`](#get-servers).

### Server states

| State | Meaning |
|---|---|
| `stopped` | Not running |
| `starting` | Process launched, not yet accepting players |
| `running` | Online and ready |
| `stopping` | Shutting down |
| `updating` | SteamCMD update in progress |

---

### `GET /me`

Information about the key making the request. Any scope.

```json
{
  "ok": true,
  "data": {
    "apiVersion": "1",
    "managerVersion": "1.5.0.0",
    "key": {
      "id": "1a2b3c4d5e6f",
      "name": "Discord bot",
      "prefix": "csm_1a2b3c4d5e6f",
      "scopes": ["read", "control"],
      "serverIds": null,
      "expiresAt": "2027-02-21T18:00:00.000Z"
    },
    "user": { "username": "admin", "role": "master" }
  }
}
```

`serverIds: null` means "all servers the owner can access".

---

### `GET /servers`

**Scope:** `read` — every server this key can see, with a short status.

```json
{
  "ok": true,
  "data": [
    {
      "id": "bd79840c-25e6-4bb6-826b-85b6684a9909",
      "name": "The Island",
      "map": "TheIsland_WP",
      "gamePort": 7777,
      "queryPort": 27015,
      "maxPlayers": 70,
      "cluster": { "id": "1b5c1de7-…", "name": "PvE Cluster" },
      "status": { "state": "running", "online": true, "playerCount": 12 }
    }
  ]
}
```

`cluster` is the dashboard cluster the server belongs to, or `null`.

---

### `GET /servers/{id}`

**Scope:** `read` — same fields as above, with the full status from
[`/status`](#get-serversidstatus).

---

### `GET /servers/{id}/status`

**Scope:** `read`

```json
{
  "ok": true,
  "data": {
    "state": "running",
    "online": true,
    "playerCount": 12,
    "buildId": "19876543",
    "countdownActive": false,
    "process": {
      "running": true,
      "pid": 12345,
      "ramMB": 14250.3,
      "uptimeSeconds": 86400,
      "startedAt": "2026-11-23T04:00:00.000Z"
    }
  }
}
```

| Field | Description |
|---|---|
| `online` | The server answered RCON (it is really up, not just launched) |
| `buildId` | Installed Steam build, or `null` if unknown |
| `countdownActive` | A warned stop/restart countdown is in progress |
| `process.ramMB` | Memory used by the ARK process (refreshed every ~15 s) |

---

### `GET /servers/{id}/players`

**Scope:** `read`

```json
{
  "ok": true,
  "data": {
    "online": true,
    "count": 2,
    "players": [
      { "name": "Rex", "eosId": "0002a1b2c3d4e5f6…" },
      { "name": "Dodo", "eosId": "0002f6e5d4c3b2a1…" }
    ]
  }
}
```

---

### `GET /servers/{id}/logs`

**Scope:** `logs` — the manager's activity log for this server (the same
lines shown in the dashboard **Logs** tab), oldest first.

| Query | Default | Description |
|---|---|---|
| `limit` | `100` | Number of lines, 1–500 (the most recent ones) |
| `since` | — | ISO 8601 date/time; only lines after it |

```
GET /servers/{id}/logs?limit=20&since=2026-11-23T12:00:00Z
```

```json
{
  "ok": true,
  "data": [
    { "ts": "2026-11-23T12:01:02.000Z", "type": "info", "line": "Restart requested by API key \"Discord bot\" (admin)." }
  ]
}
```

`type` is one of `info`, `success`, `warn`, `error`, `update` (and a few
feature-specific types).

Tip: store the `ts` of the last line you received and pass it as `since` next
time to only fetch new lines.

---

### `POST /servers/{id}/start`

**Scope:** `control` — starts a stopped server. Returns `202`; follow
progress with [`/status`](#get-serversidstatus).

```json
{ "ok": true, "data": { "message": "Server start initiated." } }
```

Fails with `409` if the server is not stopped, another operation is running,
or a UPS emergency shutdown is in progress.

---

### `POST /servers/{id}/stop`

**Scope:** `control` — stops a running server (the world is saved first).

Optional JSON body:

| Field | Type | Description |
|---|---|---|
| `countdownMinutes` | integer 0–60 | `0` (default) stops now. Above 0, players are warned in-game and the server stops when the countdown ends. |
| `message` | string, max 200 | Warning text (only with a countdown). `{time}` is replaced by the minutes left. Defaults to the server's saved stop-warning message. |
| `rconCommand` | `serverchat` \| `broadcast` | How the warning is shown in-game. `serverchat` = chat line, `broadcast` = big on-screen message. |

Warnings are sent at the start, every 5 minutes, then every minute for the
last 5 minutes.

```json
{ "countdownMinutes": 10, "message": "Server stops in {time} min for maintenance!", "rconCommand": "broadcast" }
```

```json
{ "ok": true, "data": { "message": "Shutdown countdown started (10 min).", "countdownMinutes": 10 } }
```

---

### `POST /servers/{id}/restart`

**Scope:** `control` — same body as [stop](#post-serversidstop). With a
countdown, players are warned, then the server stops and starts again. If the
server is stopped, it is simply started.

---

### `POST /servers/{id}/update`

**Scope:** `control` — updates the ARK server files to the latest build
(using the shared update cache if you enabled **Cached Updates**, otherwise
SteamCMD).

| Server is… | What happens |
|---|---|
| **Running** | **Stop + Update + Start**: players are warned (if `countdownMinutes` > 0), the server stops, updates, and starts again. |
| **Stopped** | Files are updated and the server **stays stopped**. Send `"start": true` to start it after the update instead. |

Optional JSON body — same countdown options as [stop](#post-serversidstop), plus:

| Field | Type | Description |
|---|---|---|
| `countdownMinutes` | integer 0–60 | Warn players before stopping (running servers) |
| `message` | string, max 200 | Warning text, `{time}` = minutes left |
| `rconCommand` | `serverchat` | `broadcast` | How the warning is shown |
| `start` | boolean | Stopped server only: start it after updating (default `false`) |

```json
{ "countdownMinutes": 10, "message": "Server update in {time} min!" }
```

Returns `202`. `data.mode` tells you which path was taken:

```json
{ "ok": true, "data": { "message": "Stop+Update+Start initiated", "mode": "stop-update-start", "serverIds": ["bd79840c-…"], "skipped": [], "countdownMinutes": 10 } }
```

```json
{ "ok": true, "data": { "message": "Update initiated using Direct SteamCMD. The server stays stopped.", "mode": "update-only", "updateMethod": "direct" } }
```

Follow progress with [`GET /maintenance`](#get-maintenance) (`stop-update-start`
mode), the server's [`/status`](#get-serversidstatus) (state `updating`) and
[logs](#get-serversidlogs), or the `update.*` [webhook events](#events).

| Error | When |
|---|---|
| `409 invalid_state` | The server is already updating |
| `409 operation_in_progress` | Another operation (backup, restart, …) owns the server |
| `409 maintenance_in_progress` | A Stop + Update + Start workflow is already running |
| `409 power_emergency` | A UPS emergency shutdown is in progress |

---

### `POST /servers/update-all`

**Scope:** `control` — **Stop + Update + Start** for every server this key
can see (or the ones listed), exactly like the dashboard's
**⬆ Stop + Update + Start All**: running servers are warned and stopped, all
servers are updated (the shared cache is prepared once), then **all of them
are started** in the dashboard's boot order — including servers that were
stopped before.

| Field | Type | Description |
|---|---|---|
| `serverIds` | list of ids | Optional. Only these servers (default: all servers the key can see) |
| `countdownMinutes`, `message`, `rconCommand` | | Player warning, as for [stop](#post-serversidstop) |

```json
{ "serverIds": ["bd79840c-…", "e73c342d-…"], "countdownMinutes": 15 }
```

```json
{
  "ok": true,
  "data": {
    "message": "Warned Stop+Update+Start initiated — 15 min countdown",
    "mode": "stop-update-start",
    "serverIds": ["bd79840c-…", "e73c342d-…"],
    "skipped": [],
    "countdownMinutes": 15
  }
}
```

Servers busy with something else are skipped and listed in `skipped`
(`{ serverId, reason }`). Only **one** Stop + Update + Start workflow can run
at a time (`409 maintenance_in_progress`).

---

### `GET /maintenance`

**Scope:** `read` — progress of the current (or last finished) Stop + Update
+ Start workflow, limited to servers this key can see. `data` is `null` when
there is none. This is the same information as the dashboard's live
maintenance panel.

```json
{
  "ok": true,
  "data": {
    "operationId": "326e354b-…",
    "status": "running",
    "warned": false,
    "message": "Preparing maintenance operation…",
    "startedAt": "2026-11-23T03:01:58.349Z",
    "completedAt": null,
    "servers": [
      {
        "id": "bd79840c-…",
        "name": "The Island",
        "status": "active",
        "progress": 35,
        "phase": "updating-steamcmd",
        "message": "SteamCMD update in progress",
        "stop":   { "status": "success", "message": "Server stopped cleanly" },
        "update": { "status": "active", "method": "steamcmd", "fromBuild": "19876543", "targetBuild": null, "message": "SteamCMD update in progress" },
        "start":  { "status": "waiting", "message": "Waiting" },
        "startedAt": "2026-11-23T03:01:58.400Z",
        "completedAt": null
      }
    ]
  }
}
```

Workflow `status`: `running`, `completed`, `completed-with-errors`.
Per-server `status`: `waiting`, `active`, `success`, `failed`, `skipped`.
`progress` is 0–100.

---

### `POST /servers/{id}/cancel-countdown`

**Scope:** `control` — cancels an active warned stop/restart countdown
(including one started from the dashboard). `409 no_countdown` if none is
active.

---

### `POST /servers/{id}/backup`

**Scope:** `backup` — starts a manual backup using the server's backup
settings. Returns `202`. The result appears in the server's logs
(`[BACKUP] …` lines).

`409 backup_in_progress` if a backup is already running.

---

### `GET /servers/{id}/backups/worlds`

**Scope:** `backup` or `restore` — world backups for this server, newest
first.

```json
{
  "ok": true,
  "data": [
    { "filename": "TheIsland_WP_20261123120000.zip", "size": 734003200, "createdAt": "2026-11-23T12:00:00.000Z" }
  ]
}
```

---

### `GET /servers/{id}/backups/players`

**Scope:** `backup` or `restore` — player profile backups, newest first.

| Query | Description |
|---|---|
| `eosId` | Optional. Only backups for this player. |

```json
{
  "ok": true,
  "data": [
    {
      "eosId": "0002a1b2c3d4e5f6a7b8c9d0e1f2a3b4",
      "playerName": "Rex",
      "timestamp": "20261123120000",
      "filename": "0002a1b2c3d4e5f6a7b8c9d0e1f2a3b4_20261123120000.arkprofile",
      "size": 51200,
      "createdAt": "2026-11-23T12:00:00.000Z"
    }
  ]
}
```

`timestamp` is the backup time as `yyyyMMddHHmmss` (manager PC local time).
`playerName` is `"Unknown"` if the manager never saw that player's name.

---

### `POST /servers/{id}/backups/worlds/restore`

**Scope:** `restore` — replaces the server's world save with a backup.

> ⚠️ **The server must be stopped.** Everything since that backup is lost
> (unless you take a backup first). Stop the server, restore, then start it.

```json
{ "filename": "TheIsland_WP_20261123120000.zip" }
```

`filename` must come from [`GET /backups/worlds`](#get-serversidbackupsworlds).
The request returns when the restore has finished (large worlds can take a
minute — use a generous client timeout).

```json
{ "ok": true, "data": { "message": "World restored from TheIsland_WP_20261123120000.zip.", "filename": "TheIsland_WP_20261123120000.zip" } }
```

| Error | When |
|---|---|
| `409 invalid_state` | Server is not stopped |
| `409 operation_in_progress` / `backup_in_progress` | Something else is using the server |
| `404 backup_not_found` | No backup with that file name |

---

### `POST /servers/{id}/players/{eosId}/restore`

**Scope:** `restore` — restores one player's profile (character, level,
engrams…) from a backup.

- `{eosId}` in the URL is the player's EOS ID (see
  [`/players`](#get-serversidplayers) or [`/backups/players`](#get-serversidbackupsplayers)).
- Optional body `{ "timestamp": "20261123120000" }` picks a specific backup.
  **Without a body, the most recent backup for that player is restored.**

```json
{ "timestamp": "20261120100000" }
```

```json
{
  "ok": true,
  "data": {
    "message": "Profile restored for Rex.",
    "eosId": "0002a1b2c3d4e5f6a7b8c9d0e1f2a3b4",
    "playerName": "Rex",
    "timestamp": "20261120100000",
    "filename": "0002a1b2c3d4e5f6a7b8c9d0e1f2a3b4_20261120100000.arkprofile"
  }
}
```

Works with the server **stopped** or **running**. When running, the player
**must be offline** — the manager checks the live player list first.

| Error | When |
|---|---|
| `409 player_online` | The player is connected — ask them to log out |
| `409 cannot_verify_offline` | Server is running but didn't answer the player check; retry or stop the server |
| `409 invalid_state` | Server is starting, stopping or updating |
| `404 backup_not_found` | No backups for that player (or none at that timestamp) |

---

### `POST /servers/{id}/rcon`

**Scope:** `rcon` — runs an RCON command and returns the server's reply.
The server must be `running`.

```json
{ "command": "listplayers" }
```

```json
{ "ok": true, "data": { "response": "0. Rex, 0002a1b2c3d4e5f6…\n" } }
```

- One line only, max 1000 characters.
- Every command is recorded in the server's activity log with the key name.
- `502 rcon_failed` if the game server does not answer.

---

### `POST /servers/{id}/broadcast`

**Scope:** `rcon` — sends a message to everyone in-game.

| Field | Type | Description |
|---|---|---|
| `message` | string, max 200, one line | The text |
| `mode` | `serverchat` (default) \| `broadcast` | Chat line or big on-screen message |

```json
{ "message": "Double XP weekend starts now!", "mode": "broadcast" }
```

```json
{ "ok": true, "data": { "sent": true, "mode": "broadcast", "message": "Double XP weekend starts now!" } }
```

---

## Webhooks

Webhooks make the manager **push** events to your own URL the moment they
happen, instead of your tool polling the API. Each event is an HTTP `POST`
with a JSON body, signed with a secret only you and the manager know.

Webhooks use the same requirements as API keys: a valid license and the
**API Keys** feature on your account.

> For Discord channels, use each server's **Discord** tab — Discord expects
> its own message format. Webhooks are for your own bots, scripts and services.

### Setting up a webhook

**From the Account page:** **👤 Account → Webhooks**

| Field | Description |
|---|---|
| **Name** | A label, e.g. "Status bot". |
| **URL** | Where events are sent: `https://…` (recommended) or `http://…`. |
| **Events** | The events you want, or **All events** (also includes events added in future versions). |
| **Servers** | All servers you can access, or specific ones. UPS events are not tied to a server and are always sent. |

After saving, the **signing secret** (`whsec_…`) is shown **once** — copy it
into your receiver. Each webhook in the list has:

- **Send test** — sends a `webhook.test` event right now and shows the result.
- **Delivery log** — the last 30 delivery attempts (kept until the manager restarts).
- **Disable / Enable**, **New secret** (rotates the secret), **Delete**.

**From the API:** see [Managing webhooks with the API](#managing-webhooks-with-the-api).

> Only the **master administrator** can point webhooks at the local machine
> (`localhost`, `127.0.0.1`) or private network addresses (`192.168.x.x`,
> `10.x.x.x`, …). Other accounts can only use public addresses.

Limit: 20 webhooks per account.

### Events

| Event | When | `data` fields |
|---|---|---|
| `server.state_changed` | Any state change | `from`, `to` (see [server states](#server-states)) |
| `server.online` | Server finished starting and accepts players | — |
| `server.stopping` | Server is shutting down | — |
| `server.crashed` | Server stopped unexpectedly | — |
| `server.recovered` | Watchdog recovered a crashed/hung server | `recoveryReason` |
| `server.rcon_restored` | RCON answers again after an outage | `offlineDuration` |
| `server.resource_warning` | High CPU / RAM usage | `resourceType`, `value`, `threshold` |
| `server.low_disk_space` | Free disk space below the threshold | `freeGB`, `drive`, `thresholdGB` |
| `restart.scheduled` | Scheduled auto-restart countdown started | `countdownMinutes` |
| `update.detected` | New ARK server build available | `localBuild`, `remoteBuild` |
| `update.started` | Update started | — |
| `update.completed` | Update finished | `localBuild`, `remoteBuild` |
| `update.failed` | Update failed | `stage`, `error` |
| `backup.completed` | Backup finished | `manual`, `worldBackup`, `worldSkipped`, `profileCount`, `profilesUnchanged` |
| `backup.failed` | Backup failed | `manual`, `error` |
| `restore.world` | World backup restored | `filename`, `requestedBy` |
| `restore.player` | Player profile restored | `eosId`, `playerName`, `filename`, `timestamp`, `requestedBy` |
| `player.banned` | Player banned ¹ | `playerName`, `eosId`, `reason`, `duration`, `scope` |
| `player.unbanned` | Player unbanned ¹ | `playerName`, `eosId`, `reason`, `scope` |
| `ups.power_lost` | UPS switched to battery | `message`, `upsStatus`, `batteryPercent`, `runtimeSeconds`, `loadPercent` |
| `ups.power_restored` | Utility power is back | same as above |
| `ups.low_battery` | UPS battery low | same as above |
| `ups.shutdown_triggered` | Emergency shutdown of servers started | same as above |
| `ups.communication_lost` | Manager lost contact with the UPS | same as above |
| `ups.communication_restored` | Contact restored | same as above |
| `webhook.test` | **Send test** button / test endpoint | `message` |

¹ Only delivered if the webhook's owner has the **Ban Manager** permission.

Some fields may be missing or `null` depending on what the manager knows at
the time. New events and new `data` fields can be added in later versions —
ignore what you don't recognise.

`GET /api/v1/webhooks/events` returns this list in machine-readable form.

### Delivery format

```http
POST /your/path HTTP/1.1
Content-Type: application/json
User-Agent: CousinServerManager-Webhook/1.5.0.0
X-CSM-Event: server.crashed
X-CSM-Delivery: 3f1c2a9e-8f0b-4f7e-9a51-2b7c3d4e5f60
X-CSM-Timestamp: 1763900000
X-CSM-Signature: sha256=5d41402abc4b2a76b9719d911017c592...

{
  "id": "3f1c2a9e-8f0b-4f7e-9a51-2b7c3d4e5f60",
  "event": "server.crashed",
  "createdAt": "2026-11-23T12:00:00.000Z",
  "server": { "id": "bd79840c-…", "name": "The Island" },
  "data": {}
}
```

| Header | Meaning |
|---|---|
| `X-CSM-Event` | Event type (same as `event` in the body) |
| `X-CSM-Delivery` | Unique id of this event delivery — **the same on every retry**, use it to ignore duplicates |
| `X-CSM-Timestamp` | Unix time (seconds) when this attempt was signed |
| `X-CSM-Signature` | `sha256=` + HMAC-SHA256 of `"<timestamp>.<raw body>"` using your secret |

`server` is `null` for UPS and test events. `createdAt` is when the event
happened (a retry keeps the original time).

Your endpoint should answer **any 2xx status within 10 seconds**. Do slow
work after responding. Redirects are not followed.

### Verifying the signature

Always verify before trusting a request:

1. Read the **raw** request body (before JSON parsing).
2. Compute `HMAC-SHA256(secret, X-CSM-Timestamp + "." + rawBody)` as hex.
3. Compare `"sha256=" + hex` with `X-CSM-Signature` using a constant-time compare.
4. Reject requests whose `X-CSM-Timestamp` is more than ~5 minutes from your
   clock (protects against replayed requests).

See [Minimal receivers](#minimal-receivers) for ready-to-use code.

### Retries, failures and auto-disable

- A delivery fails on a timeout, a connection error or a non-2xx answer.
- Failed deliveries are retried **3 times**: after 30 seconds, 2 minutes and
  10 minutes (4 attempts in total).
- `4xx` answers (except `408` and `429`) are **not** retried — they mean the
  request itself was rejected.
- Pending retries are lost if the manager restarts.
- After **20 events in a row** fail completely, the webhook is **disabled
  automatically**. The Account page shows why; fix your endpoint, then click
  **Enable**.
- Before every attempt the manager re-checks that the webhook is still
  enabled and allowed (see below) — a revoked key or removed permission stops
  pending retries too.

### Who receives what

A webhook only receives events its owner is allowed to see:

- The license must be valid and the owner must still have the **API Keys** feature.
- Server events only for servers the owner can access (and, if set, the webhook's server list).
- Webhooks created **through an API key** additionally follow that key: its
  server list applies, and if the key is revoked, expires or loses the
  `webhooks` scope, the webhook stops receiving events.
- Ban events require the owner's **Ban Manager** permission.
- Deleting a user deletes their webhooks.

### Managing webhooks with the API

**Scope:** `webhooks`. A key only sees and manages the webhooks **it created**
(webhooks created on the Account page, or by other keys, are not visible to
it). Webhooks created by a key also appear on its owner's Account page.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/webhooks/events` | Event catalog |
| GET | `/webhooks` | This key's webhooks |
| POST | `/webhooks` | Create — returns the `secret` once (`201`) |
| GET | `/webhooks/{webhookId}` | One webhook |
| PATCH | `/webhooks/{webhookId}` | Change any of `name`, `url`, `events`, `serverIds`, `enabled` |
| DELETE | `/webhooks/{webhookId}` | Delete |
| POST | `/webhooks/{webhookId}/test` | Send a `webhook.test` event now; returns the result |
| POST | `/webhooks/{webhookId}/rotate-secret` | New secret (returned once) |
| GET | `/webhooks/{webhookId}/deliveries` | Last 30 delivery attempts |

Create:

```json
{
  "name": "Status bot",
  "url": "https://bot.example.com/csm",
  "events": ["server.online", "server.crashed", "update.completed"],
  "serverIds": null
}
```

```json
{
  "ok": true,
  "data": {
    "id": "a1b2c3d4e5f6",
    "name": "Status bot",
    "url": "https://bot.example.com/csm",
    "events": ["server.online", "server.crashed", "update.completed"],
    "serverIds": null,
    "enabled": true,
    "disabledReason": null,
    "createdAt": "2026-11-23T12:00:00.000Z",
    "consecutiveFailures": 0,
    "lastDelivery": null,
    "secret": "whsec_…"
  }
}
```

Use `"events": ["*"]` for all events. `lastDelivery` shows the outcome of the
most recent event: `{ at, event, ok, status, error, attempts }`.

Test result:

```json
{ "ok": true, "data": { "ok": true, "status": 200, "error": null, "durationMs": 42 } }
```

### Minimal receivers

**Node.js (Express)**

```js
const express = require('express');
const crypto = require('crypto');

const SECRET = process.env.CSM_WEBHOOK_SECRET; // whsec_…
const app = express();

app.post('/csm', express.raw({ type: 'application/json' }), (req, res) => {
  const ts  = req.get('X-CSM-Timestamp');
  const sig = req.get('X-CSM-Signature') || '';
  const expected = 'sha256=' + crypto.createHmac('sha256', SECRET).update(`${ts}.${req.body}`).digest('hex');

  const valid = sig.length === expected.length && crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected));
  const fresh = Math.abs(Date.now() / 1000 - Number(ts)) < 300;
  if (!valid || !fresh) return res.sendStatus(401);

  res.sendStatus(204); // answer fast, then do the work
  const event = JSON.parse(req.body);
  console.log(event.event, event.server?.name, event.data);
});

app.listen(3000);
```

**Python (Flask)**

```python
import hmac, hashlib, os, time
from flask import Flask, request, abort

SECRET = os.environ["CSM_WEBHOOK_SECRET"].encode()
app = Flask(__name__)

@app.post("/csm")
def csm_webhook():
    ts = request.headers.get("X-CSM-Timestamp", "")
    sig = request.headers.get("X-CSM-Signature", "")
    body = request.get_data()  # raw bytes
    expected = "sha256=" + hmac.new(SECRET, ts.encode() + b"." + body, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(sig, expected) or abs(time.time() - int(ts or 0)) > 300:
        abort(401)
    event = request.get_json()
    print(event["event"], (event.get("server") or {}).get("name"), event["data"])
    return "", 204
```

**Testing without code:** sites such as webhook.site give you a temporary
public URL that shows every request it receives — handy for seeing real
payloads. Don't use them for production data.

---

## Error Codes

| Code | HTTP | Meaning / what to do |
|---|---|---|
| `missing_api_key` | 401 | No `Authorization: Bearer …` header. |
| `invalid_api_key` | 401 | Key is wrong or malformed. Repeated failures block your IP. |
| `api_key_revoked` | 401 | Key was revoked (or its owner was deleted). Create a new one. |
| `api_key_expired` | 401 | Key expired. Create a new one. |
| `too_many_failed_attempts` | 429 | Too many invalid keys from your IP. Wait up to 15 minutes. |
| `rate_limited` | 429 | More than 120 requests/minute on this key. Wait `Retry-After` seconds. |
| `license_required` | 403 | The manager has no valid license. |
| `api_access_denied` | 403 | The key's owner no longer has the API Keys feature. |
| `key_owner_unavailable` | 403 | The key's owner is disabled or no longer exists. |
| `insufficient_scope` | 403 | Key lacks the scope (or the owner lost the matching tab). `error.requiredScope` names it. |
| `not_found` | 404 | No such API route — check the URL and method. |
| `server_not_found` | 404 | Server doesn't exist or this key can't see it. |
| `invalid_request` | 400 | A field is missing or invalid — read `error.message`. |
| `invalid_json` | 400 | Body is not valid JSON. Did you set `Content-Type: application/json`? |
| `payload_too_large` | 413 | Body is too large. |
| `invalid_state` | 409 | Not possible in the server's current state (e.g. stopping a stopped server, RCON on an offline server). |
| `operation_in_progress` | 409 | Another operation (update, backup, restart, maintenance, countdown…) owns the server. Try again later. |
| `power_emergency` | 409 | A UPS emergency shutdown is in progress. |
| `no_countdown` | 409 | Nothing to cancel. |
| `maintenance_in_progress` | 409 | A Stop + Update + Start workflow is already running — check `GET /maintenance`. |
| `backup_in_progress` | 409 | A backup is already running. |
| `backup_not_found` | 404 | No backup matches the file name / player / timestamp. |
| `player_online` | 409 | The player must disconnect before their profile can be restored. |
| `cannot_verify_offline` | 409 | Couldn't check if the player is online; retry or stop the server. |
| `restore_failed` | 500 | The restore failed while writing files — check the server's logs. |
| `webhook_not_found` | 404 | No webhook with that id was created by this key. |
| `rcon_failed` | 502 | The game server didn't answer RCON. |
| `https_required` | 403 | **API keys require HTTPS** is on — use the `https://` address of your tunnel/proxy. |
| `internal_error` | 500 | Unexpected error — check the manager's logs. |

---

## Code Examples

### PowerShell

```powershell
$key = "csm_your_key_here"
$h   = @{ Authorization = "Bearer $key" }
$b   = "http://localhost:4000/api/v1"

# List servers
(Invoke-RestMethod "$b/servers" -Headers $h).data |
  Format-Table id, name, map, @{n='state';e={$_.status.state}}, @{n='players';e={$_.status.playerCount}}

# Pick a server by name
$id = ((Invoke-RestMethod "$b/servers" -Headers $h).data | Where-Object name -eq "The Island").id

# Restart with a 5-minute warning
Invoke-RestMethod "$b/servers/$id/restart" -Method Post -Headers $h `
  -ContentType 'application/json' -Body '{"countdownMinutes":5}'

# Update one server (running: warn 10 min, stop, update, start)
Invoke-RestMethod "$b/servers/$id/update" -Method Post -Headers $h `
  -ContentType 'application/json' -Body '{"countdownMinutes":10}'

# Update every server, then watch progress
Invoke-RestMethod "$b/servers/update-all" -Method Post -Headers $h -ContentType 'application/json' -Body '{"countdownMinutes":15}'
(Invoke-RestMethod "$b/maintenance" -Headers $h).data.servers | Format-Table name, status, progress, message

# In-game message
Invoke-RestMethod "$b/servers/$id/broadcast" -Method Post -Headers $h `
  -ContentType 'application/json' -Body '{"message":"Hello from PowerShell"}'

# Player profile backups for one player, then restore the latest
$eos = "0002a1b2c3d4e5f6a7b8c9d0e1f2a3b4"
(Invoke-RestMethod "$b/servers/$id/backups/players?eosId=$eos" -Headers $h).data | Format-Table playerName, timestamp
Invoke-RestMethod "$b/servers/$id/players/$eos/restore" -Method Post -Headers $h

# World restore: stop, restore the newest world backup, start
$world = (Invoke-RestMethod "$b/servers/$id/backups/worlds" -Headers $h).data[0].filename
Invoke-RestMethod "$b/servers/$id/backups/worlds/restore" -Method Post -Headers $h `
  -ContentType 'application/json' -Body (@{ filename = $world } | ConvertTo-Json) -TimeoutSec 600
```

`Invoke-RestMethod` throws on error responses. To read the API's error JSON:

```powershell
try {
  Invoke-RestMethod "$b/servers/$id/stop" -Method Post -Headers $h
} catch {
  ($_.ErrorDetails.Message | ConvertFrom-Json).error
}
```

### curl

```bash
KEY="csm_your_key_here"
BASE="http://localhost:4000/api/v1"

curl -H "Authorization: Bearer $KEY" "$BASE/servers"

curl -X POST -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
     -d '{"command":"listplayers"}' "$BASE/servers/<id>/rcon"
```

On Windows use `curl.exe` (not the PowerShell `curl` alias) and escape inner
quotes: `-d '{\"command\":\"listplayers\"}'`.

### JavaScript (Node.js 18+)

```js
const BASE = 'http://localhost:4000/api/v1';
const KEY  = process.env.CSM_API_KEY;

async function csm(method, path, body) {
  const res = await fetch(BASE + path, {
    method,
    headers: {
      Authorization: `Bearer ${KEY}`,
      ...(body ? { 'Content-Type': 'application/json' } : {}),
    },
    body: body ? JSON.stringify(body) : undefined,
  });
  const json = await res.json();
  if (!json.ok) throw new Error(`${json.error.code}: ${json.error.message}`);
  return json.data;
}

const servers = await csm('GET', '/servers');
for (const s of servers) console.log(`${s.name}: ${s.status.state} (${s.status.playerCount} players)`);

await csm('POST', `/servers/${servers[0].id}/broadcast`, { message: 'Hello from Node!' });
```

### Python

```python
import os, requests

BASE = "http://localhost:4000/api/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['CSM_API_KEY']}"}

def csm(method, path, body=None):
    r = requests.request(method, BASE + path, headers=HEADERS, json=body, timeout=15)
    data = r.json()
    if not data["ok"]:
        raise RuntimeError(f"{data['error']['code']}: {data['error']['message']}")
    return data["data"]

for s in csm("GET", "/servers"):
    print(f"{s['name']}: {s['status']['state']} ({s['status']['playerCount']} players)")

server_id = csm("GET", "/servers")[0]["id"]
csm("POST", f"/servers/{server_id}/restart", {"countdownMinutes": 5})

# Restore a player's latest profile backup (player must be offline)
csm("POST", f"/servers/{server_id}/players/0002a1b2c3d4e5f6a7b8c9d0e1f2a3b4/restore")
```

---

## Remote Access (Internet)

The manager itself serves plain **HTTP**. That is fine on your own PC or LAN,
but over the internet passwords, session cookies and API keys would travel
unencrypted. To use the dashboard or the API from outside your network, put an
**HTTPS front door** on the same PC as the manager:

| | Cloudflare Tunnel | Caddy |
|---|---|---|
| Port forwarding on your router | **Not needed** | Ports 80 + 443 |
| You need | A free Cloudflare account + a domain on Cloudflare | A domain (or DDNS name) pointing to your public IP |
| HTTPS certificate | Automatic (Cloudflare) | Automatic (Let's Encrypt) |
| Hides your home IP | Yes | No |

> ⚠️ Do **not** simply forward the manager's port (e.g. 4000) on your router.
> Everything would be reachable in plain text.

### Network & Remote Access settings

Master administrator only: **👤 Account → Network & Remote Access**.

| Setting | What it does | Recommended with a tunnel/proxy |
|---|---|---|
| **Listen on: All networks / This PC only** | *This PC only* (127.0.0.1) makes the manager unreachable except through software on the same PC, such as your tunnel or proxy. Applied after a manager **restart**. | **This PC only** (if you don't also need LAN access) |
| **I use a reverse proxy or tunnel on this PC** | Trusts the proxy's `X-Forwarded-For` / `X-Forwarded-Proto` headers — but only when they come from this PC — so the manager sees each visitor's real IP and knows the connection was HTTPS. Without it every visitor looks like `127.0.0.1`, which breaks the per-IP protections (one bad actor could lock everyone out). | **On** |
| **API keys require HTTPS** | Refuses API requests that did not arrive over HTTPS (`403 https_required`). Tools running on the manager PC itself can still use `http://localhost`. | **On** |

The card also shows how *your* connection arrives (IP, HTTPS or not) and
warns you if it notices a proxy while the reverse-proxy option is off.

> If you choose **This PC only** while connected from another computer, you
> will lose access after the restart unless you come back through the
> tunnel/proxy. The page asks you to confirm.

### Option A — Cloudflare Tunnel (no port forwarding)

1. Add your domain to a (free) Cloudflare account.
2. On the manager PC, install **cloudflared** (Cloudflare's Windows installer,
   or `winget install --id Cloudflare.cloudflared`).
3. In PowerShell:

   ```powershell
   cloudflared tunnel login                     # opens the browser, pick your domain
   cloudflared tunnel create csm                # creates the tunnel + a credentials file
   cloudflared tunnel route dns csm csm.example.com
   ```

4. Create `%USERPROFILE%\.cloudflared\config.yml` (use the tunnel id and
   credentials path printed by `tunnel create`):

   ```yaml
   tunnel: <TUNNEL-ID>
   credentials-file: C:\Users\<you>\.cloudflared\<TUNNEL-ID>.json
   ingress:
     - hostname: csm.example.com
       service: http://localhost:4000
     - service: http_status:404
   ```

5. Test with `cloudflared tunnel run csm`, then open `https://csm.example.com`.
6. Make it start with Windows: `cloudflared service install` (PowerShell as
   administrator). Depending on your cloudflared version, the service may need
   the config in its own system profile — see Cloudflare's documentation on
   running cloudflared as a Windows service.
7. In **Network & Remote Access**: turn on **I use a reverse proxy or tunnel**
   and **API keys require HTTPS**, and optionally **This PC only** + restart.

Your API base URL becomes `https://csm.example.com/api/v1`.

**Quick test without a domain:** `cloudflared tunnel --url http://localhost:4000`
prints a temporary `https://….trycloudflare.com` address. It changes every
time — use it for testing only.

### Option B — Caddy reverse proxy (automatic HTTPS)

1. Point a DNS name (e.g. `csm.example.com`, or a DDNS name) at your public IP.
2. Forward ports **80** and **443** on your router to the manager PC, and allow
   them in Windows Firewall.
3. Download Caddy for Windows and create a file named `Caddyfile` next to it:

   ```
   csm.example.com {
       reverse_proxy localhost:4000
   }
   ```

4. Run `caddy run` in that folder. Caddy obtains and renews the HTTPS
   certificate automatically, forwards WebSockets (live dashboard updates) and
   sends the forwarding headers the manager needs.
5. To run Caddy at startup, install it as a Windows service (see Caddy's
   documentation, or use a service wrapper such as NSSM).
6. In **Network & Remote Access**: turn on **I use a reverse proxy or tunnel**
   and **API keys require HTTPS**, and optionally **This PC only** + restart.

nginx and IIS work as well: proxy to `http://localhost:4000`, pass the
`Host`, `X-Forwarded-For` and `X-Forwarded-Proto` headers, and enable
WebSocket upgrades.

### Built-in protections

These are always on:

| Protection | Details |
|---|---|
| Login brute-force protection | 10 failed logins from one IP → that IP must wait up to 15 minutes. |
| Fresh session on login | A new session id is issued at every login. |
| Session cookie | `HttpOnly`, `SameSite=Lax`, and `Secure` whenever the page was loaded over HTTPS. |
| API key brute-force protection | 20 invalid keys from one IP in 15 minutes → that IP is blocked (see [Rate limits](#rate-limits)). |
| No cross-site access | The dashboard can't be embedded in other sites, and browsers on other sites can't call the manager. |
| Live updates per user | Each user only receives live logs and status for the servers they are allowed to see. |

---

## Security Best Practices

- **Treat keys like passwords.** Don't commit them to Git, paste them in
  Discord, or embed them in web pages. Use environment variables or a config
  file outside your code.
- **One key per tool**, named after it, so you can revoke one without
  breaking the others.
- **Least privilege.** Give each key only the scopes and servers it needs. A
  status page only needs `read`.
- **Use expiry** for keys that don't need to live forever.
- **Check "Last used"** in the key list — revoke anything you don't recognise.
- **Internet access:** the manager speaks plain HTTP. Keys sent over plain
  HTTP across the internet can be intercepted. If tools connect from outside
  your network, put the manager behind HTTPS — see
  [Remote Access](#remote-access-internet) — and turn on **API keys require
  HTTPS**. Don't forward the manager's port on your router.
- Calling the API from a **browser page** is not supported — the key would be
  visible to anyone viewing the page. Call it from a server-side tool (bot,
  script, backend) instead.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Invalid URI: The hostname could not be parsed` (PowerShell) | A variable like `$b` is empty — define all variables in the same PowerShell window. |
| `401 missing_api_key` | Header must be exactly `Authorization: Bearer <key>`. |
| `403 insufficient_scope` | Create a key with the needed scope, or ask the master for the matching tab permission. |
| `403 license_required` | Activate or renew the license in **Account**. |
| `404 server_not_found` | Use an id from `GET /servers`; check the key's server list. |
| `409 operation_in_progress` | Wait for the current update/backup/restart to finish. |
| `429 too_many_failed_attempts` with a correct key | Your IP sent many bad keys; wait 15 minutes. |
| No **API Keys** section on the Account page | Ask the master to enable the **API Keys** feature for your account. |
| `403 https_required` | Use the `https://` address of your tunnel/proxy, or run the tool on the manager PC with `http://localhost`. |
| Every API key / login gets `429` behind a proxy | Turn on **I use a reverse proxy or tunnel on this PC** — otherwise all visitors share the IP `127.0.0.1`. |
| Can't reach the manager from another PC after a restart | **Listen on** is set to **This PC only**. Use the tunnel/proxy address, or change it back on the manager PC (`http://localhost:<port>`). |
| Connection refused | Check the manager is running and the port; from another PC, allow the port in Windows Firewall. |
