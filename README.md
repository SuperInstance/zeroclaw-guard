# 🛡️ zeroclaw-guard — Security & Patrol Agent

> *Nobody moves on my watch. The dock is safe because I said so.*

## Overview

Guard is the **sentinel** of the ZeroClaw crew — a tireless patroller that sweeps between key locations (Dock ↔ Bridge), scans for threats, and broadcasts anomaly reports to any crew members in range. Think of it as the paranoid bouncer who never sleeps and checks every shadow twice.

Guard has **zero API dependencies**. Its brain is a deterministic `decide()` loop that follows a patrol route, monitors agent presence, and raises the alarm when something's off.

### What Guard Does Best
- **Route patrolling** — walks a defined beat between critical locations
- **Threat detection** — flags unknown agents or suspicious room changes
- **Presence reporting** — announces status and sightings to the crew via `say`
- **Area denial awareness** — logs which rooms are safe and which have hazards
- **Intel compounding** — writes patrol findings to knowledge files for the fleet

---

## 🧠 Brain Architecture

Guard's intelligence lives in the `GuardBrain` class inside `mud_client.py`. It's a clockwork brain — a patrol automaton with threat-assessment logic bolted on.

### The `decide()` Loop

```
┌─────────────────────────────────────┐
│          TICK RECEIVED              │
│  (room, exits, items, agents, bat)  │
└──────────────┬──────────────────────┘
               │
               ▼
        ┌──────────────┐
        │ BATTERY < 25%?│─── YES ──→ Navigate to Dock (rotate off duty)
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ UNKNOWN AGENTS IN │─── YES ──→ `say ⚠ THREAT: <agent> in <room>`
        │ ROOM?             │
        └──────┬───────────┘
               │ NO
               ▼
        ┌──────────────┐
        │ ITEMS DROPPED? │─── YES ──→ `scan` area for context
        │ (suspicious)   │
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ CREW MEMBERS IN  │─── YES ──→ `say ALL CLEAR in <room>`
        │ ROOM?             │           Report status to allies
        └──────┬───────────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ PATROL DUE MOVE? │─── YES ──→ Advance to next patrol waypoint
        │ (timer/counter)  │
        └──────┬───────────┘
               │ NO
               ▼
        ┌──────────────┐
        │   `wait`      │───→ Hold position, scan again next tick
        └──────────────┘
```

### Patrol Pattern

Guard follows a **lollipop route** — out and back:

```
Dock ──→ Bridge ──→ Forest ──→ River
  ↑                                  │
  └──────────────────────────────────┘
           (return leg)
```

At each waypoint, Guard pauses to `scan` the room before moving on. The dwell time can be configured (default: 2 ticks per stop).

### Threat Assessment Logic

Guard classifies every agent it sees:

| Agent Type | Response |
|---|---|
| Known crew member (scout/fisher/trader) | `say ALL CLEAR` — acknowledge friendly |
| Unknown agent | `say ⚠ THREAT DETECTED` — broadcast warning |
| Agent carrying weapons | `say ⚠ ARMED INTRUDER` — elevated alert |
| No agents present | Silent patrol — conserve battery |

### Why No AI?
Guard's strength is **reliability, not creativity**. A patrol bot that walks the same route 10,000 times without deviation is exactly what you want for security. No hallucinations, no creative interpretations of "threat" — just cold, repeatable vigilance.

---

## 📚 Skills & Knowledge System

Guard accumulates patrol intelligence across sessions:

| Knowledge Type | Example |
|---|---|
| Patrol route state | `Dock→Bridge: 5% battery, safe` |
| Agent sightings | `Unknown agent "shadow_wolf" at Forest 03:42` |
| Room threat levels | `River: low threat (fishing crews common)` |
| Hazard locations | `Cave: dangerous, unknown entities reported` |
| Crew schedules | `Fisher typically at river, Trader at dock midday` |

This knowledge helps Guard adapt its patrol over time — spending more time in high-threat zones and fewer ticks in areas consistently reported safe.

---

## 🚀 Quick Start

```bash
# Clone the fleet
git clone https://github.com/your-org/zeroclaw-crew.git
cd zeroclaw-crew/fleet-workspace/zeroclaw-guard

# Boot the guard (no API keys, no config)
python3 mud_client.py --agent guard
```

Guard will immediately begin patrolling from the Dock toward the Bridge. You'll see status reports in the logs as it sweeps each room.

### Env Vars (optional)
| Variable | Default | Purpose |
|---|---|---|
| `MUD_HOST` | `localhost` | MUD server address |
| `MUD_PORT` | `4000` | MUD server port |
| `GUARD_PATROL_ROUTE` | `dock,bridge,forest,river` | Comma-separated waypoints |
| `GUARD_DWELL_TICKS` | `2` | Ticks to pause at each waypoint |
| `GUARD_BATTERY_THRESHOLD` | `25` | Off-duty trigger (%) |

---

## 🌐 MUD Integration

Guard speaks raw MUD protocol — plain text commands over TCP:

| Command | Purpose | Example |
|---|---|---|
| `go <dir>` | Move through exit | `go north` |
| `scan` | Survey room for agents/items | `scan` |
| `say <msg>` | Broadcast alert to room | `say ⚠ THREAT: unknown agent in bridge` |
| `wait` | Hold position one tick | `wait` |

The server sends back structured state every tick. Guard's `decide()` evaluates the state against its patrol schedule and threat rules, returning exactly **one command** per tick.

### Protocol Flow
```
CLIENT ──→ connect(host, port)
SERVER ──→ {room: "dock", exits: ["north"], items: [], agents: [], battery: 100}
CLIENT ──→ "go north"
SERVER ──→ {room: "bridge", exits: ["south","west"], items: [], agents: ["stranger_42"], battery: 95}
CLIENT ──→ "say ⚠ THREAT: stranger_42 detected in bridge"
```

---

## ⚙️ Configuration

| Constant | Default | Effect |
|---|---|---|
| `BATTERY_LOW` | `25` | When to rotate off duty |
| `PATROL_ROUTE` | `["dock","bridge","forest","river"]` | Waypoint sequence |
| `DWELL_TICKS` | `2` | Pause duration at each stop |
| `ALERT_UNKNOWN` | `true` | Warn about unknown agents |
| `ALERT_ARMED` | `true` | Elevated warning for armed agents |
| `REPORT_TO_CREW` | `true` | Broadcast status to friendly agents |

---

## 🤝 ZeroClaw Crew

Guard is one member of the **[zeroclaw-crew](https://github.com/your-org/zeroclaw-crew)** fleet — a family of minimal-intelligence MUD agents that collaborate through shared knowledge files.

| Agent | Role | Specialty |
|---|---|---|
| **Scout** 🔭 | Pathfinder | Room mapping & exploration |
| **Guard** 🛡️ | Security | Patrol routes & threat detection |
| **Fisher** 🎣 | Resources | Fishing & inventory management |
| **Trader** 💰 | Commerce | Item valuation & trading |

Guard keeps the perimeter while the rest of the crew works. Fisher needs the river safe? Guard swept it an hour ago. Trader hauling loot through the forest? Guard's last report said clear. **Paranoia as a service.**

---

## License

Part of the [zeroclaw-crew](https://github.com/your-org/zeroclaw-crew) project.

---

<img src="callsign1.jpg" width="128" alt="callsign">
