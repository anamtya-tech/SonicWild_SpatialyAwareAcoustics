# 🦜 SonicWild
> **For:** Collaborators/New SWE Interns joining the SonicWild project  
> **Goal:** Understand everything about this application — what it does, how it's built, and how all the pieces connect — in simple language.

---

## 📋 Table of Contents

1. [What is SonicWild? (The Big Picture)](#1-what-is-sonicwild)
2. [The Problem It Solves](#2-the-problem-it-solves)
3. [System Architecture — How all layers connect](#3-system-architecture)
4. [The Hardware — SonicWild Nodes](#4-the-hardware--SonicWild-nodes)
5. [The Backend — server.js explained](#5-the-backend)
6. [The Frontend — React App explained](#6-the-frontend)
7. [How Data Flows — End to End](#7-how-data-flows-end-to-end)
8. [The Database](#8-the-database)
9. [Every Page Explained](#9-every-page-explained)
10. [Key Concepts You Must Know](#10-key-concepts-you-must-know)
11. [Tech Stack Cheat Sheet](#11-tech-stack-cheat-sheet)
12. [REST API Quick Reference](#12-rest-api-quick-reference)
13. [How to Run the App (Dev Mode)](#13-how-to-run-the-app-dev-mode)
14. [File Map — Where is what?](#14-file-map--where-is-what)
15. [Git & Branching](#15-git--branching)
16. [Testing](#16-testing)
17. [Common Gotchas](#17-common-gotchas)
18. [Glossary](#18-glossary)
19. [Month-over-Month Changes](#19-month-over-month-changes)

---

## 1. What is SonicWild?

> **One line:** SonicWild is a Windows desktop app that listens to forests and tells you — in real time — **exactly where in spacital space** every sound is coming from.

Imagine you're a wildlife researcher deep in a tropical forest. You've placed 4 small microphone boxes around the area. An owl hoots somewhere above you at 45 degrees, 20 meters away. SonicWild shows you that on a spacital map — the exact angle, elevation, and direction of that sound — **live**, as it happens.

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   🌳  Forest Environment                               │
│                                                         │
│   🎙️ Mic Node A         🎙️ Mic Node B                  │
│         \                    /                          │
│          \    🦜 Bird       /                          │
│           \   Sound Here  /                             │
│            \_____________/                              │
│                   │                                     │
│                   ↓ (WiFi HotSpot- no internet)         │
│          💻 SonicWild Laptop                           │
│          Shows: "Sound at 45°, 20m high, NE direction"  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Who uses it?
- 🧑‍🔬 Wildlife researchers and ecologists
- 🌿 Conservation NGOs
- 🎓 University bioacoustics research groups
- 🔧 Field technicians deploying sensors

---

## 2. The Problem It Solves

Human ears can hear a sound but can't precisely tell you:
- **Where exactly** it came from (spacital position)
- **How high up** it was (elevation)
- **Which species** (needs AI classification)
- **Multiple sounds simultaneously** at different directions

SonicWild solves this with a **network of microphone arrays** that do math (beamforming) to figure out the spacital origin of every sound.

```
WITHOUT SonicWild:
  Researcher hears frog → writes in notebook "heard frog, somewhere to the left"

WITH SonicWild:
  Researcher sees on screen → "Frog call: Azimuth 270°, Elevation 15°, 
                                confirmed stable source, 0.87 activity level"
```

---

## 3. System Architecture

SonicWild runs as **4 layers stacked together**:

```
╔══════════════════════════════════════════════════════════╗
║  LAYER 1: HARDWARE (in the forest)                       ║
║                                                          ║
║  🍓 SonicWild Node (Raspberry Pi)                        ║
║   • 4-microphone array attached                          ║
║   • Runs ODAS software (beamforming math)                ║
║   • Sends sound data over WiFi - hotspot (no internet)   ║
║   • Accessible via SSH for config/control                ║
╠══════════════════════════════════════════════════════════╣
║  LAYER 2: BACKEND (server.js — Node.js)                  ║
║                                                          ║
║  • Receives TCP streams from hardware devices            ║
║  • Exposes REST API on port 3000                         ║
║  • Pushes real-time data to frontend via Socket.IO        ║
║  • Manages SSH connections to devices                    ║
║  • Reads/writes the file-based database                  ║
╠══════════════════════════════════════════════════════════╣
║  LAYER 3: FRONTEND (React + TypeScript)                  ║
║                                                          ║
║  • Shows 2D maps with sound direction rays               ║
║  • Renders spacital sphere showing elevation + azimuth         ║
║  • Displays real-time spectrograms (frequency graphs)    ║
║  • Provides all UI controls for the researcher           ║
╠══════════════════════════════════════════════════════════╣
║  LAYER 4: ELECTRON SHELL                                 ║
║                                                          ║
║  • Wraps everything into a .exe desktop application      ║
║  • Starts the backend and frontend automatically         ║
║  • No browser needed — it IS its own browser             ║
╚══════════════════════════════════════════════════════════╝
```

### How they connect:

```
SonicWild Node ──TCP:9000──► server.js ──Socket.IO──► React UI
                │                                      │
                └──SSH──► config files                 │
                                                       ▼
                                              Three.js spacital sphere
                                              Leaflet 2D map
                                              Spectrogram Renderer
```

---

## 4. The Hardware — Chatak Nodes

These are the physical devices placed in the forest.

```
┌─────────────────────────────────────┐
│        CHATAK NODE                  │
│  ┌─────────────────────┐            │
│  │  Raspberry Pi       │            │
│  │  (small computer)   │            │
│  └────────┬────────────┘            │
│           │                         │
│  ┌────────▼────────────┐            │
│  │  4-Channel Mic Hat  │ ←── 4 mics │
│  └─────────────────────┘            │
│                                     │
│  Software Running:                  │
│  • ODAS (audio processing)          │
│  • ChatakGUI (SonicWild client)     │
│                                     │
│  Config Files at:                   │
│  /home/chatak/ChatakGUI/config/     │
│  ├── deviceId.txt    (unique ID)    │
│  ├── project_name.txt               │
│  ├── record.txt      (record mins)  │
│  ├── pause.txt       (pause mins)   │
│  ├── wake.txt        (wake time)    │
│  ├── sleep.txt       (sleep time)   │
│  ├── latlong.txt     (location)     │
│  ├── passive_config.txt (ODAS cfg)  │
│  └── mac_id.txt      (device MAC)   │
└─────────────────────────────────────┘

Default SSH Login:
  Username: chatak
  Password: chatak
```

### What ODAS does (simplified):
ODAS (Open embeddeD Audition System) is free open-source software running ON the Raspberry Pi. It:
1. Captures audio from all 4 microphones simultaneously
2. Does **beamforming** — math that compares tiny time differences between mics to figure out which direction sound came from
3. Outputs the direction as **X, Y, Z direction cosines** (basically a spacital arrow pointing toward the sound)
4. Categorizes sources as **Tracks** (stable, confirmed) or **Pots** (potential/noisy)
5. Streams this data as JSON over TCP at ~60 updates per second

---

## 5. The Backend

**File:** `server.js` — about 4,077 lines and 147KB — the backbone of everything.

### What server.js does:

```
server.js responsibilities:
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. REST API (Express on port 3000)                 │
│     • /sync-values → SSH to device, read config    │
│     • /updateSettings → push settings via SSH      │
│     • /list-odas-config-files → list remote cfgs   │
│     • /save-mic-location → persist GPS coordinates │
│     • /api/devices → discover devices on network   │
│     • /projects, /mic-locations → SQLite CRUD      │
│                                                     │
│  2. TCP Listener (per device, port 8081+)           │
│     • Opens raw TCP socket to listen to ODAS data  │
│     • Custom parser: reads raw bytes → JSON objects │
│     • Forwards parsed data to Socket.IO             │
│                                                     │
│  3. Socket.IO Server (same port 3000)               │
│     • Pushes events to connected frontend           │
│     • Events: track_info, pot_info, triangulated..  │
│                                                     │
│  4. SSH Bridge (node-ssh)                           │
│     • Connects to Chatak nodes                      │
│     • Reads/writes config files                     │
│     • Starts/stops ODAS/audio workflows remotely    │
│                                                     │
│  5. Network Scanner (via src/scanner.js)            │
│     • Pings every IP on local subnet                │
│     • SSH-interrogates responding IPs               │
│     • Finds which ones are Chatak nodes             │
└─────────────────────────────────────────────────────┘
```

### Supporting backend files:
| File | What it does |
|------|-------------|
| `main.js` | Electron entry — launches backend and opens the desktop window |
| `preload.js` | Electron security bridge for renderer access |
| `database.js` | Primary SQLite runtime using `better-sqlite3` |
| `database_manager.js` | Legacy TXT database helper retained for compatibility |
| `src/scanner.js` | Network device discovery (ping sweep + SSH interrogation) |
| `src/scanner-api.js` | SSE wrapper around scanner/discovery |
| `src/interrogateDevice.js` | SSH helpers to verify Chatak devices |

---

## 6. The Frontend

**Location:** `frontend/src/`

The frontend is a **React 18 + TypeScript** app built with Vite. Think of it as a very complex web app running inside Electron.

### React App structure (from App.tsx upward):

```
main.tsx  ← React starts here (like index.js in other apps)
    │
    ▼
App.tsx   ← Sets up all global "wrappers" (Providers) + URL routing
    │
    ├── QueryClientProvider    → manages API data fetching/caching
    ├── DeviceProvider         → stores "which device is selected"
    ├── SidebarProvider        → stores sidebar open/closed state
    ├── HashRouter             → URL navigation (#/live, #/setup, etc.)
    │
    └── Routes (URL → Page mapping):
      /                     → DeviceConnection.tsx  (home screen)
      /device/:ip           → DeviceSettingsPage.tsx (per-device settings)
      /dashboard            → Dashboard.tsx         (overview)
      /device-connection    → DeviceConnection.tsx  (explicit route alias)
      /setup                → Setup.tsx             (deployment/setup wizard)
      /project-setup/dashboard → ProjectDashboardPageAdvanced.tsx

      ///Legacy Pages:
      /project-setup/list   → ProjectListPage.tsx
      /live                 → LiveTriangulationAdvanced.tsx (main live view)
      /live-monitoring      → LiveMonitoring.tsx    (monitoring view)
      /live-triangulation   → LiveTriangulationAdvanced.tsx
      /live-triangulation-advanced → LiveTriangulationAdvanced.tsx
      /archive              → ArchiveViewer.tsx     (browse recordings)
      /manage-classifier    → ManageClassifierPage.tsx
      *                     → NotFound.tsx          (catch-all)
```

### Services (how frontend talks to backend):

```
frontend/src/services/
├── api.ts              → Makes HTTP calls to server.js (GET/POST/DELETE)
├── socket.ts           → Socket.IO connection for real-time track/pot data
├── visualizationSocket.ts → Dedicated visualization socket helpers
└── dataSimulator.ts    → Local/mock data support for development
```

### Hooks (reusable logic shared across pages):

```
frontend/src/hooks/
├── useRealTimeData.ts      → Subscribes to track_info & pot_info events
├── useTriangulationData.ts → Subscribes to triangulatedPosition events
├── useDeviceManagement.ts  → Manages connected device list and settings actions
├── useSoundLines.ts        → Converts ODAS data to map lines
├── use-mobile.tsx          → Responsive/mobile helper
└── use-toast.ts            → Toast notification helper
```

### Key Visualization Libraries:

```
Three.js      → spacital sphere showing sound in X/Y/Z space
               (the "sound bubble" hemisphere)


Leaflet       → 2D interactive map with offline tile support
               (the map showing mic positions + sound rays)

Recharts      → Time-series charts and statistics graphs
```

---

## 7. How Data Flows End to End

### Flow A: Sound Data (Live Monitoring)

```
🎙️ Forest Sound (e.g., bird call)
         │
         ▼
🍓 SonicWild Node (Raspberry Pi)
   ODAS captures from 4 mics simultaneously
   Beamforming math → figures out direction
   Output: { src: [{id:55,tag:real,x:0.7, y:0.3, z:0.6, activity:0.85,class:frog,conf:0.45}] }
         │
         │  TCP connection (raw bytes, port 3000)
         │  Continuous stream, ~60 updates/second
         ▼
💻 server.js (TCP Listener)
   Receives raw bytes
   Custom parser assembles complete JSON objects
   (handles fragmented packets from network)
         │
         ▼
📡 Socket.IO (inside server.js)
   Emits "track_info" event to all connected frontends
   Emits "pot_info" event for potential sources
         │
         │  WebSocket connection (port 9002/9003)
         ▼
⚛️ React Frontend
   useRealTimeData hook receives the events
   Passes data to map components, spacital sphere, spectrogram
         │
         ▼
🖥️ Rendered on Screen:
   • Blue ray on map (direction of stable sound track)
   • Dot on spacital sphere (azimuth + elevation)
   • Spike in spectrogram (frequency content)
```

### Flow B: Device Discovery

```
👆 User clicks "Scan for Devices"
         │
         ▼
GET /api/devices (SSE stream endpoint)
         │
         ▼
src/scanner.js runs
   Pings every IP on local network (e.g., 192.168.1.1 to .254)
   For responding IPs → SSH attempt
   Checks if /home/chatak/ChatakGUI/ exists
   Reads deviceId.txt to confirm it's a Chatak node
         │
         │  Results stream back as Server-Sent Events
         ▼
DeviceConnection.tsx
   Shows device cards in real-time as they're found
   User enters SSH credentials (chatak/chatak)
   Clicks "Connect"
```

### Flow C: Push Settings to Device

```
👆 User changes recording/config settings in UI
         │
         ▼
POST /updateSettings { ip, recordMin, pauseMin, wakeTime, sleepTime, ... }
         │
         ▼
server.js uses node-ssh:
   Connects to device via SSH (chatak@192.168.1.x)
   Calls /home/chatak/ChatakGUI/update_settings.py
   Updates config TXT files and optional passive_config.txt
   Preserves selected values when optional args are marked KEEP
         │
         ▼
✅ Device is now recording with new settings
```

---

## 8. The Database

SonicWild now runs primarily on **SQLite** using `better-sqlite3`.

**Primary file:** `zodas.db`  
**Runtime manager:** `database.js`  
**Legacy helper:** `database_manager.js`

Current SQLite schema includes tables such as:

```
projects
sublocations
devices
device_config
mic_locations
monitoring_sessions
audio_files
inference_results
app_metadata
```

> **Note:** `zodas_database.txt` still exists as a legacy artifact/reference, but current backend runtime operates in SQLite mode.

### Where is it stored?
- **Development:** project root / user data path as `zodas.db`
- **Production (installed app):** writable app data path resolved by `SONICWILD_USER_DATA`

---

## 9. Every Page Explained

### 🔌 Page 1: Device Connection (`/`)
**File:** `frontend/src/pages/DeviceConnection.tsx`

```
PURPOSE: Find and connect to Chatak hardware nodes on the network

┌─────────────────────────────────────────────────┐
│  Device Connection                               │
│                                                  │
│  [🔍 Scan Network]  [+ Add Manually: 192.168.x] │
│                                                  │
│  Found Devices:                                  │
│  ┌──────────────────────────────────────────┐   │
│  │ 🟢 chatak-01  │ 192.168.1.101  │ Online  │   │
│  │ 🟡 chatak-02  │ 192.168.1.102  │ Pinging │   │
│  │ 🔴 chatak-03  │ 192.168.1.103  │ Offline │   │
│  └──────────────────────────────────────────┘   │
│                                                  │
│  Selected: chatak-01                            │
│  SSH User: [chatak] Password: [chatak]          │
│  [Connect & Sync Config]                        │
└─────────────────────────────────────────────────┘
```

**What happens when you click "Connect":**
1. POST /sync-values → SSH into device → reads config files
2. Device state saved locally
3. TCP listener started for this device's data stream

---

### ⚙️ Page 2: Setup Wizard (`/setup - Legacy Page`)
**File:** `frontend/src/pages/Setup.tsx`

```
PURPOSE: Define your research deployment before going live

This screen is no longer best understood as a standalone wizard.
In the current frontend, the app is organized around **two major navigation links**:

1. **Devices**
2. **Projects**

The old “many primary pages” model is outdated. Most important workflows now branch from one of these two entry points.

**Key data this page saves:**
- Base camp GPS (where the laptop is)
- Each microphone's GPS position
- Recording window in minutes
- Species target (frogs / birds / primates)
- Project name

---

### 📱 Current Navigation Model

#### 1. Devices (`/` and `/device-connection`)
**Primary files:** `frontend/src/pages/DeviceConnection.tsx`, `frontend/src/pages/DeviceSettingsPage.tsx`, `frontend/src/components/DevicePopup.tsx`

This is the **hardware-facing** side of the app.

Use it to:
- discover Chatak nodes on the network
- connect over SSH
- open the per-device popup for live control and monitoring
- open the per-device settings page
- do bulk updates across connected nodes
- manage Wi-Fi, ODAS config file selection, mic location, and archive access per device

The Devices Area: 
- SSH flows
- `/sync-values`
- `/updateSettings`
- live device health and telemetry
- popup-driven audio/ODAS controls

#### 2. Projects (`/project-setup/list`)
**Primary files:** `frontend/src/pages/ProjectListPage.tsx`, `frontend/src/pages/ProjectDashboardPageAdvanced.tsx`, `frontend/src/pages/Setup.tsx`

This is the **deployment and planning** side of the app.

Use it to:
- create and delete projects
- manage project sublocations
- define deployment geometry and map context
- connect mics to sublocations
- move into live or recorded workflows from a project context

Projects now act as the organizer for most field operations. The older guide’s “independent page list” is less useful than understanding that **projects define context**, while **devices execute and report**.

---

### 📡 Project-Driven Live Views

Once a project or sublocation is selected, the app launches one of the monitoring/analysis views below.

#### Live Triangulation (`/live`, `/live-triangulation`, `/live-triangulation-advanced`)
**File:** `frontend/src/pages/LiveTriangulationAdvanced.tsx`

```
PURPOSE: Primary live spatial monitoring screen for project-linked deployments

┌────────────────────────────────────────────────────────┐
│                    LIVE TRIANGULATION                  │
│                                                        │
│  ┌───────────────────┐  ┌──────────────────────────┐   │
│  │                   │  │    spacital HEMISPHERE   │   │
│  │   2D MAP          │  │                          │   │
│  │   (Leaflet/GMaps) │  │    [sphere diagram]      │   │
│  │                   │  │    Elevation 0-90°       │   │
│  │  🎙️──────►  (blue ray = track)                 │   │
│  │  🎙️ ···►    (red ray = pot)                    │   │
│  │                   │  │    Azimuth 0-360°        │   │
│  └───────────────────┘  └──────────────────────────┘   │
│                                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │  SPECTROGRAM (frequency heatmap 0-20kHz)        │   │
│  │  [colorful waterfall chart going right in time] │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│  Devices: [Node-01 🟢] [Node-02 🟢]  [⏸ Pause]   │
└────────────────────────────────────────────────────────┘
```

Key behaviors:
- 🔵 **Blue rays** = **Tracks** (stable, confirmed sound sources — ODAS is confident)
- 🔴 **Red rays** = **Pots** (potential / noisy / brief detections — less confident)
- consumes live device streams and triangulated positions
- shows map-based spatial context plus real-time acoustic visualization

---

#### Live Monitoring (`/live-monitoring`)
**File:** `frontend/src/pages/LiveMonitoring.tsx`

```
PURPOSE: Operational live-observation screen for mic-centric monitoring, streaming, and triangulation review

Features:
• Loads deployed mic/base-camp context from saved project/device state
• Shows 2D map with live rays and optional triangulation points
• Supports per-mic hemishpehe viewer launch/stop
• Supports live audio streaming from a selected device
• Includes spectrogram/audio controls and triangulation test mode helpers
• Shows device telemetry such as battery, signal, memory, and online state
```

---

#### Archive Viewer (`/archive`)
**File:** `frontend/src/pages/ArchiveViewer.tsx`

```
PURPOSE: Archive-analysis surface for replay-style visualization of recorded sessions

Current state in code:
• lets the user choose project, device, and recording label from UI controls
• provides playback-style controls (play, pause, seek/slider)
• renders archived tracks/pots into the SoundHemisphere view
• includes chart/control sidebars for archive inspection
• currently uses placeholder/mock recording lists and simulated archived data derived from current track/pot state

Important note:
• The richer SSH-driven archive management flow exists elsewhere in the codebase (for example in popup/archive manager components), while `ArchiveViewer.tsx` itself is currently a lighter archive playback UI rather than the full remote-file browser described in older docs.
```

---

#### Dashboard (`/dashboard`)
**File:** `frontend/src/pages/Dashboard.tsx`

```
PURPOSE: Quick overview of everything

Shows:
• high-level project/device snapshot
• quick navigation back into major workflows
• summary-style operational context
```

---

### Project Management Screens

#### Project List (`/project-setup/list`)
**File:** `frontend/src/pages/ProjectListPage.tsx`

```
PURPOSE: Entry point for the Projects side of the app

• List all projects
• Create new project records
• Open a selected project dashboard
• Delete stale project records
```

---

#### Project Dashboard (`/project-setup/dashboard`)
**File:** `frontend/src/pages/ProjectDashboardPageAdvanced.tsx`

```
PURPOSE: Main project operations hub

• Manage sublocations within a project
• Configure live vs recorded deployment contexts
• Link microphones to sublocations
• Launch live monitoring from a chosen sublocation
• Maintain project-level deployment geometry and map context
```


```
PURPOSE: Guided deployment configuration screen used within the Projects workflow

• place base camp / hotspot context
• set microphone positions on map
• define project-linked session parameters
• prepare device-side settings before live or recorded operation
```

---



---

## 10. Key Concepts You Must Know

### What is a Track vs a Pot?

```
ODAS Data:
┌─────────────────────────────────────────────────┐
│                                                 │
│  "src" (Sources) = TRACKS                       │
│  • Stable, persistent sound sources             │
│  • ODAS tracked it for multiple frames          │
│  • High confidence = "yes, this is a real sound"│
│  • Shown as BLUE rays on the map                │
│                                                 │
│  "pot" (Potentials) = POTS                      │
│  • Short-lived detections                       │
│  • Could be noise, reflection, brief sound      │
│  • Lower confidence = "might be something"      │
│  • Shown as RED rays on the map                 │
│                                                 │
└─────────────────────────────────────────────────┘
```

### What are Direction Cosines?

ODAS doesn't give you "45 degrees north-east". It gives you `x, y, z` values.

```
Direction Cosines (x, y, z):
• These are the components of a unit vector
• A unit vector always has: x² + y² + z² = 1
• Example: { x: 0.707, y: 0.707, z: 0 } = sound at 45° horizontal, no elevation
• SonicWild converts these to human-friendly Azimuth + Elevation angles

Conversion:
  Azimuth   = atan2(y, x)    → horizontal angle (0-360°)
  Elevation = asin(z)        → vertical angle (0-90°)
```

### What is Beamforming?

```
4 mics in a square arrangement:

    🎙️ Mic1        🎙️ Mic2
         \              /
          \   Sound    /
           \  arrives /
            \_  at  _/
           🎙️ Mic3  🎙️ Mic4

Each mic hears the sound at a SLIGHTLY different time.
The time differences between mics = tells you the direction.

This is "beamforming" — mathematically "steering" toward the source.
ODAS does all this math inside the Raspberry Pi.
```

### What is Socket.IO vs REST API?

```
REST API (HTTP):
  • For one-time requests
  • Frontend asks → Backend answers → done
  • Examples: "Save this project", "Get device list"
  • Slow back-and-forth cycle

Socket.IO (WebSocket):
  • For continuous real-time streaming
  • Backend pushes data to frontend WITHOUT being asked
  • Examples: track_info, pot_info (60 times per second!)
  • Like a radio that's always broadcasting

SonicWild uses BOTH:
  REST API → configuration, projects, archive listings
  Socket.IO → live acoustic data (tracks, pots, triangulation)
```

---

## 11. Tech Stack Cheat Sheet

### Backend (server.js)
| What | Technology | Why |
|------|-----------|-----|
| HTTP server | **Express.js** | Industry standard Node.js web framework |
| Real-time streaming | **Socket.IO** | WebSocket wrapper — easy rooms + events |
| SSH to Raspberry Pi | **node-ssh** | Connects to devices remotely |
| Raw device data | **net (TCP)** | Node.js built-in — reads ODAS byte streams |
| Desktop wrapper | **Electron** | Packages web app as .exe |
| Database | **File (TXT)** | Simple, portable, no database server needed |

### Frontend (React)
| What | Technology | Why |
|------|-----------|-----|
| UI framework | **React 18** | Component-based UI, very popular |
| Type safety | **TypeScript** | Catches bugs before they run |
| Build tool | **Vite** | Very fast dev server + bundler |
| Styling | **Tailwind CSS** | Utility classes (no custom CSS writing) |
| UI components | **shadcn/ui** | Pre-built accessible components |
| State management | **TanStack Query** | Caches API responses, auto-refetches |
| URL routing | **React Router** | Handles /live, /setup, etc. |
| spacital rendering | **Three.js** | WebGL spacital — renders the sound hemisphere |
| 2D Maps | **Leaflet** | Interactive maps (offline capable) |
| Online Maps | **Google Maps API** | When internet available |
| Icons | **Lucide** | Clean icon set |

---

## 12. REST API Quick Reference

Base URL: `http://localhost:3000`

### Device & SSH
| Method | URL | What it does |
|--------|-----|-------------|
| POST | `/sync-values` | SSH into device, read current settings |
| POST | `/updateSettings` | Push new settings to device via SSH |
| POST | `/list-odas-config-files` | List `.cfg` files on a remote device |
| POST | `/run-command` | Execute shell command on device |
| GET | `/api/devices` | SSE stream — discover devices on network |

### Projects & Data
| Method | URL | What it does |
|--------|-----|-------------|
| POST | `/save-project` | Create/update a research project |
| GET | `/projects` | Get all projects |
| DELETE | `/project/:id` | Delete a project |
| POST | `/save-mic-location` | Save mic GPS position |
| GET | `/mic-locations` | Get all mic locations |

### Maps
| Method | URL | What it does |
|--------|-----|-------------|
| GET | `/load-tile` | Serve offline map tile image |
| GET | `/maps/manifest` | List available offline map bundles |

### Archives
| Method | URL | What it does |
|--------|-----|-------------|
| GET | `/archive/list` | List recording files on a device (via SSH) |
| GET | `/archive/download` | Download a recording from a device |

### Real-Time Events (Socket.IO)
| Event | Direction | Contains |
|-------|-----------|---------|
| `track_info` | Server → Browser | Array of stable sound sources |
| `pot_info` | Server → Browser | Array of potential detections |
| `triangulatedPosition` | Server → Browser | spacital position `{x, y, z, lat, lng}` |
| `device_status` | Server → Browser | Device went online/offline |

---

## 13. How to Run the App (Dev Mode)

```bash
# Step 1: Install root (backend/Electron) dependencies
npm install

# Step 2: Install frontend dependencies
cd frontend
npm install
cd ..

# Step 3: Start the backend (runs on port 3000)
node server.js

# Step 4: Start the frontend dev server
cd frontend
npm run dev

# Step 5: If you want the backend-served production bundle refreshed
# npm run build:frontend
```

### Ports to know:
| Port | What's running |
|------|---------------|
| 3000 | Backend API + Socket.IO (server.js) |
| 8080 | Frontend dev server (React/Vite) |
| 8081+ | Per-device TCP listeners (one per Chatak node) |
| 9090/9091 | Meta Fusion Server (multi-sensor triangulation) |

---

## 14. File Map — Where is what?

```
SonicWild-3DEcoAcoustic_makparjdev/
│
│  ── BACKEND ──
├── main.js                  ← Electron start — opens the window
├── preload.js               ← Security bridge (Electron ↔ React)
├── server.js                ← ⭐ THE BRAIN — all backend logic
├── database.js              ← Primary SQLite runtime
├── database_manager.js      ← Legacy TXT compatibility helper
├── src/
│   ├── scanner.js           ← Finds Chatak devices on WiFi
│   ├── scanner-api.js       ← SSE wrapper for scanner
│   ├── interrogateDevice.js ← SSH checks if device is Chatak
│   ├── ssh_success.json     ← Cache of working IPs
│   └── ssh_failures.json    ← Cache of failing IPs
│
│  ── FRONTEND ──
├── frontend/src/
│   ├── main.tsx             ← React starts here
│   ├── App.tsx              ← Routing + global providers
│   ├── config/
│   │   └── environment.ts   ← Where is the backend? (API URL)
│   ├── services/
│   │   ├── api.ts           ← All HTTP calls (GET/POST)
│   │   └── socket.ts        ← Socket.IO connection
│   ├── hooks/
│   │   ├── useRealTimeData.ts  ← Listens for tracks/pots
│   │   ├── useTriangulationData.ts ← Listens for spacital positions
│   │   ├── useDeviceManagement.ts  ← Device management helper
│   │   └── useSoundLines.ts        ← Map line conversion helper
│   ├── pages/              ← One file = one screen
│   │   ├── DeviceConnection.tsx   ← Home / connect screen
│   │   ├── DeviceSettingsPage.tsx ← Per-device settings screen
│   │   ├── Setup.tsx              ← Project wizard
│   │   ├── LiveTriangulationAdvanced.tsx ← MAIN LIVE SCREEN
│   │   ├── LiveMonitoring.tsx     ← Monitoring screen
│   │   ├── ProjectListPage.tsx    ← Project list
│   │   ├── ProjectDashboardPageAdvanced.tsx ← Project dashboard
│   │   └── ArchiveViewer.tsx      ← Browse recordings
│   └── components/
│       ├── DevicePopup.tsx        ← Per-device control panel
│       ├── spectrogram.tsx        ← Frequency chart
│       ├── terrain/               ← OfflineLeafMap and map helpers
│       ├── visualization/         ← Viewer widgets and charts
│       └── layout/
│           └── Layout.tsx           ← App shell (sidebar + content)
│
│  ── DATABASE ──
├── zodas.db                 ← Active SQLite database
├── zodas_database.txt       ← Legacy TXT format reference
│
│  ── DOCS ──
├── README.md
├── docs/intern/SONICWILD_MASTER_CONTEXT.md    ← Deep technical doc
├── docs/intern/SONICWILD_INTERN_GUIDE.md      ← Intern onboarding guide
└── docs/intern/SONICWILD_KNOWLEDGE_BASE_FOR_RAG.md
```

---

## 15. Git & Branching

### Working with branches:
```bash
git branch -a
```

### How to work as an intern:
```bash
# Start from the branch your team is actively using
git branch --show-current
git pull

# Create your own branch for your task
git checkout -b feature/your-feature-name

# Work, then commit
git add .
git commit -m "feat: what you did"

# Push your branch
git push origin feature/your-feature-name

# Ask for review — avoid pushing directly to shared long-lived branches unless requested
```

---

## 16. Testing

```bash
# Backend unit tests
npm run test:backend

# Integration tests
npm run test:integration

# End-to-end tests (Playwright — simulates real user clicks)
npm run test:e2e

# Frontend tests
cd frontend && npm test
```

### What each tests:
| Test Type | Tests | Location |
|-----------|-------|---------|
| Backend unit | Database CRUD, API routes | `tests/backend/` |
| Integration | Device discovery, SSH flows | `tests/integration/` |
| E2E (Playwright) | Full user journeys in browser | `tests/e2e/` |
| Frontend unit | React components | `frontend/src/test/` |

---

## 17. Common Gotchas

> [!WARNING]
> **Things that trip up new interns:**

1. **"The map shows blank tiles"**
   - Online maps need a valid Google Maps API key in `.env`
   - Offline mode needs tiles in `public/maps/`
   - Solution: Download tile bundles from the Settings page first

2. **"No data showing on the live screen"**
   - Check that a Chatak device is connected and streaming
   - Check that the TCP listener started (look for port 8081+ in server logs)
   - Check that ODAS is running on the Raspberry Pi

3. **"Socket.IO not connecting"**
   - Backend must be running on port 3000 first
   - Check browser console for connection errors
   - Vite proxy in `vite.config.ts` routes `/socket.io` to port 3000

4. **"SSH connection refused"**
   - Default credentials: `chatak` / `chatak`
   - Device must be on the same WiFi network
   - Check `src/ssh_failures.json` — might be cached as failed

5. **"Changes not reflecting in frontend"**
   - Frontend dev server has Hot Module Replacement (HMR)
   - If HMR fails, do a full browser refresh
   - If backend changed, restart `node server.js`

6. **"Database not persisting"**
   - Dev mode saves to project root as `zodas_database.txt`
   - Production saves to `%APPDATA%/SonicWild/zodas_database.txt`
   - Don't delete this file accidentally!

---

## 18. Glossary

| Term | Meaning |
|------|---------|
| **ODAS** | Open embeddeD Audition System — open-source software running ON the Raspberry Pi that does the beamforming math |
| **Chatak Node** | The physical Raspberry Pi device with 4 microphones placed in the forest |
| **Track (src)** | A stable, confident sound source identified by ODAS |
| **Pot (potential)** | A brief or uncertain detection — might be real, might be noise |
| **Beamforming** | Math technique using multiple mics to determine sound direction |
| **Direction Cosine** | X, Y, Z components describing which way a sound came from |
| **Azimuth** | Horizontal angle (0-360°) — like compass bearing |
| **Elevation** | Vertical angle (0-90°) — how high up the sound is |
| **Triangulation** | Combining data from multiple nodes to pinpoint exact spacital location |
| **Socket.IO** | Library for real-time bidirectional communication (like WhatsApp for code) |
| **SSE** | Server-Sent Events — server pushes text data to browser (one-way streaming) |
| **TCP** | Transmission Control Protocol — raw network byte streaming (what ODAS uses) |
| **SSH** | Secure Shell — remote terminal access to Raspberry Pi |
| **Electron** | Framework that lets you build desktop apps using web technologies |
| **Vite** | Modern, very fast build tool for React apps |
| **shadcn/ui** | Pre-built React UI components (buttons, cards, dialogs, etc.) |
| **TanStack Query** | Library that manages API data fetching, caching, and automatic refresh |
| **Three.js** | JavaScript library for spacital graphics in the browser (used for hemisphere) || **Leaflet** | Open-source 2D map library with offline support |
| **Spectrogram** | Visual representation of sound frequencies over time (colorful heatmap) |
| **Meta Fusion Server** | C++ process on ports 9090/9091 that combines data from multiple nodes |
| **HashRouter** | React Router using `#` in URLs (e.g., `localhost:8080/#/live`) |
| **HMR** | Hot Module Replacement — Vite updates code in browser without full page reload |
| **NSIS** | Windows installer format — what `electron-builder` creates |

---

> **You've read the guide! 🎉**
> 
> **Next steps:**
> 1. Run the app in dev mode (Section 13)
> 2. Explore `frontend/src/pages/LiveTriangulationAdvanced.tsx` — the main screen
> 3. Read `server.js` top-to-bottom (the heart of the backend)
> 4. Ask a question when something doesn't make sense!

---


---

## 19. Month-over-Month Changes

This section gives a quick timeline of major changes from actual git commit history.
It is intended as a lightweight orientation aid, not an exhaustive changelog.

### 2026-07
- Device popup and live monitoring stability work (audio/spectrogram startup reliability, stop flow behavior).
- Live-session and ODAS workflow hardening across backend/frontend interactions.
- Update Settings flow improvements (config handling, KEEP sentinel path, per-device and bulk refinements).
- Documentation sync updates and cleanup commits.
- Note: This month includes many `sprint-hourly` automation commits in addition to feature/fix commits.

### 2026-05
- RAG and file manager refinements.
- Safe local audio/JSON handling improvements.

### 2026-04
- Major velocity month with broad UI/UX and workflow updates.
- SQLite migration work from TXT DB paths and related settings updates.
- Multi-device bulk update capability additions.
- Location map integration and spectrogram/hemisphere/live monitoring improvements.

### 2026-03
- Dashboard layout refinements and map control improvements.
- Roadmap/documentation additions and general frontend quality updates.

### 2026-02
- Stable baseline and README/project-structure updates.

### How to maintain this section
- Run: `git log --date=format:%Y-%m --pretty=format:"%ad|%h|%s"`
- Summarize by month, grouping noisy automation commits separately from functional changes.
