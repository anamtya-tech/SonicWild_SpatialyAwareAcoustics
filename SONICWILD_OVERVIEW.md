# 🦜 SonicWild — Intern Onboarding Guide
> **For:** New SWE Interns joining the SonicWild project  
> **Goal:** Understand everything about this application — what it does, how it's built, and how all the pieces connect — in simple language.

---

## 📋 Table of Contents

1. [What is SonicWild? (The Big Picture)](#1-what-is-sonicwild)
2. [The Problem It Solves](#2-the-problem-it-solves)
3. [System Architecture — How all layers connect](#3-system-architecture)
4. [The Hardware — Chatak Nodes](#4-the-hardware--chatak-nodes)
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
│   🎙️ Mic Node A         🎙️ Mic Node B                │
│         \                    /                          │
│          \    🦜 Bird       /                           │
│           \   Sound Here  /                             │
│            \_____________/                              │
│                   │                                     │
│                   ↓ (WiFi)                              │
│          💻 SonicWild Laptop                            │
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
║  🍓 Chatak Node (Raspberry Pi)                          ║
║   • 4-microphone array attached                          ║
║   • Runs ODAS software (beamforming math)                ║
║   • Sends sound data via TCP over WiFi                   ║
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
Chatak Node ──TCP:9000──► server.js ──Socket.IO──► React UI
                │                                      │
                └──SSH──► config files                 │
                                                       ▼
                                              Three.js spacital sphere
                                              Leaflet 2D map
                                              LightningChart spectrogram
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
│  ├── record.txt      (recording?)   │
│  └── pause.txt       (duty cycle)   │
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

**File:** [`server.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/server.js) — 3,131 lines, 120KB — the backbone of everything.

### What server.js does:

```
server.js responsibilities:
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. REST API (Express on port 3000)                 │
│     • /sync-values → SSH to device, read config    │
│     • /projects → CRUD for research projects       │
│     • /mic-locations → save GPS coordinates        │
│     • /api/devices → discover devices on network   │
│     • /archive/list → list recordings on device    │
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
│     • Runs shell commands remotely                  │
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
| [`main.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/main.js) | Electron entry — launches server.js and opens the browser window |
| [`preload.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/preload.js) | Bridge between Electron and React (security layer) |
| [`database_manager.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/database_manager.js) | Reads and writes the flat-file database |
| [`src/scanner.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/src/scanner.js) | Network device discovery (ping sweep) |
| [`src/interrogateDevice.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/src/interrogateDevice.js) | SSH into devices to verify they're Chatak nodes |

---

## 6. The Frontend

**Location:** [`frontend/src/`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/)

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
        /setup                → Setup.tsx             (configure project)
        /live                 → LiveTriangulationAdvanced.tsx (main live view)
        /live-monitoring      → LiveMonitoring.tsx    (monitoring view)
        /archive              → ArchiveViewer.tsx     (browse recordings)
        /dashboard            → Dashboard.tsx         (overview)
        /project-setup/list   → ProjectListPage.tsx
        /manage-classifier    → ManageClassifierPage.tsx
```

### Services (how frontend talks to backend):

```
frontend/src/services/
├── api.ts              → Makes HTTP calls to server.js (GET/POST/DELETE)
├── socket.ts           → Socket.IO connection for real-time track/pot data
└── visualizationSocket.ts → Second socket for triangulation visualization
```

### Hooks (reusable logic shared across pages):

```
frontend/src/hooks/
├── useRealTimeData.ts      → Subscribes to track_info & pot_info events
├── useTriangulationData.ts → Subscribes to triangulatedPosition events
├── useDeviceManagement.ts  → Manages connected device list
└── useSoundLines.ts        → Converts ODAS data to map lines
```

### Key Visualization Libraries:

```
Three.js      → spacital sphere showing sound in X/Y/Z space
               (the "sound bubble" hemisphere)

LightningChart → Real-time spectrogram (frequency heatmap)
               (the colorful waterfall chart)

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
🍓 Chatak Node (Raspberry Pi)
   ODAS captures from 4 mics simultaneously
   Beamforming math → figures out direction
   Output: { src: [{x:0.7, y:0.3, z:0.6, activity:0.85}] }
         │
         │  TCP connection (raw bytes, port 9000)
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
         │  WebSocket connection (port 3000)
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
👆 User changes "Record for 30 minutes" setting in UI
         │
         ▼
POST /updateSettings { ip, recordMin: 30, pauseMin: 5 }
         │
         ▼
server.js uses node-ssh:
   Connects to device via SSH (chatak@192.168.1.x)
   Writes new value to /home/chatak/ChatakGUI/config/record.txt
   Restarts the recording service on the device
         │
         ▼
✅ Device is now recording with new settings
```

---

## 8. The Database

SonicWild uses a **simple flat-file database** — no MySQL, no MongoDB. Just a text file!

**File:** `zodas_database.txt`  
**Manager:** [`database_manager.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/database_manager.js)

```
📄 zodas_database.txt looks like this:

[metadata]
{"version": "1.0", "schema": "sonicwild-v1"}

[projects]
[
  {
    "id": "1716900000000",
    "name": "Dhurandhar Forest Site",
    "description": "Monsoon frog survey 2026",
    "baseLocation": { "lat": 21.1, "lng": 79.0 }
  }
]

[micLocations]
[
  {
    "id": "mic_001",
    "name": "Node A - Riverbank",
    "lat": 21.101,
    "lng": 79.001,
    "deviceId": "chatak-01",
    "projectId": "1716900000000"
  }
]

[deviceConfigs]
[
  {
    "ip": "192.168.1.101",
    "userId": "chatak",
    "password": "chatak",
    "recordMin": "30",
    "pauseMin": "5"
  }
]
```

> **Note:** There's also a `zodas.db` SQLite file for a newer migration, but the primary active database is the flat text file.

### Where is it stored?
- **Development:** project root folder
- **Production (installed app):** `%APPDATA%/SonicWild/zodas_database.txt`

---

## 9. Every Page Explained

### 🔌 Page 1: Device Connection (`/`)
**File:** [`DeviceConnection.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/DeviceConnection.tsx) (53KB)

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

### ⚙️ Page 2: Setup Wizard (`/setup`)
**File:** [`Setup.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/Setup.tsx) (56KB)

```
PURPOSE: Define your research deployment before going live

Two modes:

LIVE SESSION (4 steps):        PASSIVE SESSION (3 steps):
Step 1: Kit & Coverage         Step 1: Grid Setup
   ↓ place hotspot on map         ↓ auto-calculate center
Step 2: Mic Placement          Step 2: Deployment Schedule
   ↓ drag mics to positions       ↓ set duty cycle (e.g., 10min/hour)
Step 3: Session Config         Step 3: Confirm & Deploy
   ↓ choose species target
Step 4: Confirm & Launch
   ↓ push settings to devices
```

**Key data this page saves:**
- Base camp GPS (where the laptop is)
- Each microphone's GPS position
- Recording window in minutes
- Species target (frogs / birds / primates)
- Project name

---

### 📡 Page 3: Live Triangulation (`/live`)
**File:** [`LiveTriangulationAdvanced.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/LiveTriangulationAdvanced.tsx) (102KB — the BIGGEST file!)

```
PURPOSE: The MAIN screen — shows live spacital sound positions in real time

┌─────────────────────────────────────────────────────────┐
│                    LIVE TRIANGULATION                    │
│                                                          │
│  ┌───────────────────┐  ┌──────────────────────────┐   │
│  │                   │  │    spacital HEMISPHERE          │   │
│  │   2D MAP          │  │                           │   │
│  │   (Leaflet/GMaps) │  │    [sphere diagram]       │   │
│  │                   │  │    Elevation 0-90°         │   │
│  │  🎙️──────►  (blue ray = track)                  │   │
│  │  🎙️ ···►    (red ray = pot)                    │   │
│  │                   │  │    Azimuth 0-360°          │   │
│  └───────────────────┘  └──────────────────────────┘   │
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │  SPECTROGRAM (frequency heatmap 0-20kHz)        │   │
│  │  [colorful waterfall chart going right in time] │   │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
│  Devices: [chatak-01 🟢] [chatak-02 🟢]  [⏸ Pause]   │
└─────────────────────────────────────────────────────────┘
```

**What the colors mean:**
- 🔵 **Blue rays** = **Tracks** (stable, confirmed sound sources — ODAS is confident)
- 🔴 **Red rays** = **Pots** (potential / noisy / brief detections — less confident)

---

### 📊 Page 4: Live Monitoring (`/live-monitoring`)
**File:** [`LiveMonitoring.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/LiveMonitoring.tsx) (68KB)

```
PURPOSE: Detailed monitoring view — more device controls, audio streaming

Features:
• Per-device spacital hemisphere ("sound bubble")
• Live audio streaming (hear the actual forest audio!)
• Device controls (start/stop recording)
• Real-time time-series charts
• Device telemetry (battery, temp, network signal)
```

---

### 📁 Page 5: Archive Viewer (`/archive`)
**File:** [`ArchiveViewer.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/ArchiveViewer.tsx) (15KB)

```
PURPOSE: Browse and download recordings stored on devices

┌──────────────────────────────────────────┐
│  ARCHIVE VIEWER                          │
│                                          │
│  Device: chatak-01                       │
│  ┌─────────────────────────────────────┐ │
│  │ 📄 recording_2026-06-01_08h.wav    │ │
│  │ 📄 recording_2026-06-01_09h.wav    │ │
│  │ 📄 recording_2026-06-01_10h.wav    │ │
│  └─────────────────────────────────────┘ │
│                                          │
│  [▶ Play]  [⬇ Download]  [📊 Spectrogram]│
└──────────────────────────────────────────┘

Files are fetched over SSH from the Raspberry Pi storage.
```

---

### 📋 Page 6: Dashboard (`/dashboard`)
**File:** [`Dashboard.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/Dashboard.tsx) (21KB)

```
PURPOSE: Quick overview of everything

Shows:
• How many devices are connected
• Active project name
• Recent detections summary
• Quick links to main workflows
```

---

### 📁 Page 7: Project Management (`/project-setup/list`)
**Files:** [`ProjectListPage.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/ProjectListPage.tsx), [`ProjectDashboardPageAdvanced.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/ProjectDashboardPageAdvanced.tsx)

```
PURPOSE: Organize work into named projects

• List all research projects (e.g., "Dhurandhar Survey 2026")
• Create / Edit / Delete projects
• Each project links to specific mic locations
• Per-project dashboard with statistics
```

---

### 🤖 Page 8: Manage Classifiers (`/manage-classifier`)
**File:** [`ManageClassifierPage.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/ManageClassifierPage.tsx) (13KB)

```
PURPOSE: Upload AI models to Chatak devices

• Upload BirdNET or custom TFLite models (.tflite files)
• Deploy models to specific devices via SSH
• Track which model version is on which device
• Status: Partially implemented (planned for v1.2)
```

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
| Spectrograms | **LightningChart JS** | High-performance real-time charts |
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
# Step 1: Install root (backend) dependencies
npm install

# Step 2: Install frontend dependencies
cd frontend
npm install
cd ..

# Step 3: Start the backend (runs on port 3000)
node server.js

# Step 4: Start the frontend dev server (runs on port 8080)
cd frontend
npm run dev

# Step 5: Open browser
# http://localhost:8080
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
SonicWild_Ship/
│
│  ── BACKEND ──
├── main.js                  ← Electron start — opens the window
├── preload.js               ← Security bridge (Electron ↔ React)
├── server.js                ← ⭐ THE BRAIN — all backend logic
├── database_manager.js      ← Reads/writes zodas_database.txt
├── database.js              ← SQLite alternative (newer)
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
│   │   └── useTriangulationData.ts ← Listens for spacital positions
│   ├── pages/              ← One file = one screen
│   │   ├── DeviceConnection.tsx   ← Home / connect screen
│   │   ├── Setup.tsx              ← Project wizard
│   │   ├── LiveTriangulationAdvanced.tsx ← MAIN LIVE SCREEN
│   │   ├── LiveMonitoring.tsx     ← Monitoring screen
│   │   └── ArchiveViewer.tsx      ← Browse recordings
│   └── components/
│       ├── DevicePopup.tsx        ← Per-device control panel
│       ├── spectrogram.tsx        ← Frequency chart
│       ├── visualization/
│       │   ├── SphereVisualizer.tsx ← spacital hemisphere (Three.js)
│       │   └── MapViewer.tsx        ← 2D Leaflet map
│       └── layout/
│           └── Layout.tsx           ← App shell (sidebar + content)
│
│  ── DATABASE ──
├── zodas_database.txt       ← Main database (projects, mics, devices)
├── zodas.db                 ← SQLite (newer, migration target)
│
│  ── DOCS ──
├── README.md
├── SONICWILD_MASTER_CONTEXT.md    ← Deep technical doc
├── CODEBASE_REFERENCE.md          ← Complete code reference
└── SYSTEM_INTEGRATION.md
```

---

## 15. Git & Branching

### Current branches:
```bash
git branch -a

# Main active branch:
dev                          ← Where daily development happens

# Production ready:
release-v1.1-live-export     ← Production-ready branch

# Safe restore points (tags):
v1.1-working-may29           ← ✅ Everything worked on May 29, 2026
v1.1-stable                  ← Earlier stable point
```

### How to work as an intern:
```bash
# Always start from dev branch
git checkout dev
git pull origin dev

# Create your own branch for your task
git checkout -b feature/your-feature-name

# Work, then commit
git add .
git commit -m "feat: what you did"

# Push your branch
git push origin feature/your-feature-name

# Ask for review — never push directly to dev or release
```

### If something breaks, restore to working state:
```bash
# View safe snapshot
git checkout v1.1-working-may29

# Make a new branch from it to fix issues
git checkout -b fix/restore-working v1.1-working-may29
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
| **Three.js** | JavaScript library for spacital graphics in the browser (used for hemisphere) |
| **LightningChart** | High-performance charting library (used for spectrogram) |
| **Leaflet** | Open-source 2D map library with offline support |
| **Spectrogram** | Visual representation of sound frequencies over time (colorful heatmap) |
| **Meta Fusion Server** | C++ process on ports 9090/9091 that combines data from multiple nodes |
| **HashRouter** | React Router using `#` in URLs (e.g., `localhost:8080/#/live`) |
| **HMR** | Hot Module Replacement — Vite updates code in browser without full page reload |
| **NSIS** | Windows installer format — what `electron-builder` creates |

---

> **You've read the intern guide! 🎉**
> 
> **Next steps:**
> 1. Run the app in dev mode (Section 13)
> 2. Explore [`LiveTriangulationAdvanced.tsx`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/frontend/src/pages/LiveTriangulationAdvanced.tsx) — the main screen
> 3. Read [`server.js`](file:///c:/SonicWild_Ship%20(3)/SonicWild_Ship/server.js) top-to-bottom (the heart of the backend)
> 4. Ask a question when something doesn't make sense!

---
*SonicWild Intern Guide — Written June 2026; refreshed July 2026*

---

## 19. Month-over-Month Changes

This section gives interns a quick timeline of major changes from actual git commit history.
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
