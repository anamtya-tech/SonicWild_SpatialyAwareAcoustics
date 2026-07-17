# 🦜 SonicWild — Master Project Context Document
> **Purpose:** This file is the single source of truth for any IDE, AI assistant (Claude, Copilot, Gemini, etc.), or new developer joining this project. Read this file first. It covers the mission, full architecture, technology stack, page-by-page feature breakdown, data flows, APIs, database schema, file structure, and future roadmap.

---

## 1. 🎯 What Is SonicWild?

**SonicWild** is a spatially-aware, real-time **ecoacoustic monitoring and visualization platform** built as a Windows desktop application (Electron). It connects to a network of custom field sensors called **Chatak Nodes** (Raspberry Pi-based devices running ODAS — Open embeddeD Audition System) and provides researchers with a live **spacital window into the acoustic environment** of forests, wetlands, and wildlife habitats.

### One-Sentence Pitch
> *"SonicWild is a desktop ecosystem for discovering, managing, and visualizing real-time spacital bioacoustic data from remote field sensors — designed to tell researchers not just what sounds are in an environment, but exactly where they come from in spacital space."*

### The Problem It Solves
In dense ecosystems like tropical forests, identifying the exact location and elevation of a sound (a frog call, a primate, a bird species) using human ears alone is impossible. SonicWild uses a network of **synchronized 4-channel microphone arrays** to triangulate sound positions and project them onto:
- A **2D map** (Google Maps / Leaflet) showing directional "rays" from each sensor
- A **spacital hemisphere viewer** showing Azimuth + Elevation in real-time
- A **spectrogram** showing the frequency signature of the sounds

### Who Uses It
- Wildlife researchers and ecologists
- Conservation NGOs deploying acoustic sensors in remote areas
- University research groups studying bioacoustics and soundscapes
- Field technicians managing sensor deployments

### Competitors / Inspiration
- **Synature** (synature.com) — European company doing similar bioacoustic monitoring at scale
- **Wildlife Acoustics** — AudioMoth and Song Meter recorders
- **BirdNET** — AI classification for bird sounds

SonicWild's differentiation is its **real-time spacital spatial visualization** and **offline-first field deployment** design.

---

## 2. 🏗️ System Architecture

SonicWild is a **triple-layer distributed system** running entirely on a Windows machine:

```
┌─────────────────────────────────────────────────────┐
│                  ELECTRON SHELL                      │
│  (main.js + preload.js)                              │
│  • Launches backend server                           │
│  • Hosts React frontend in BrowserWindow             │
│  • Injects runtime config (API_BASE_URL, SOCKET_URL) │
└────────────────────┬────────────────────────────────┘
                     │
     ┌───────────────┴────────────────┐
     │         BACKEND LAYER          │
     │  server.js (Node.js/Express)   │
     │  Port: 3000                    │
     │  • REST API                    │
     │  • Socket.IO server            │
     │  • TCP listeners (per device)  │
     │  • SSH bridge (node-ssh)       │
     │  • database_manager.js         │
     └───────────────┬────────────────┘
                     │
     ┌───────────────┴────────────────┐
     │         FRONTEND LAYER         │
     │  React 18 + Vite + TypeScript  │
     │  Port: 8080 (dev)              │
     │  • Pages & UI                  │
     │  • Three.js spacital rendering       │
     │  • LightningChart spectrograms │
     │  • Leaflet / Google Maps       │
     └────────────────────────────────┘
                     │
     ┌───────────────┴────────────────┐
     │         HARDWARE LAYER         │
     │  Chatak Nodes (Raspberry Pi)   │
     │  • Running ODAS                │
     │  • 4-channel microphone array  │
     │  • Emits JSON via TCP          │
     │  • Accessible via SSH          │
     └────────────────────────────────┘
```

### Key Design Principles
- **Offline-First**: Works without internet. Maps are cached locally (Leaflet tiles). No cloud dependency.
- **Portable**: The app ships as a self-contained `.exe` with a bundled Node.js runtime.
- **Real-Time**: Data flows at up to 60Hz from hardware → TCP → Socket.IO → React UI.
- **Field-Ready**: Handles intermittent WiFi, SSH timeouts, device reboots gracefully.

---

## 3. 🧰 Technology Stack

### Desktop Shell
| Technology | Role |
|---|---|
| **Electron** | Desktop container, window management, IPC |
| `main.js` | Electron entry — starts backend, creates BrowserWindow |
| `preload.js` | Exposes safe APIs to renderer via contextBridge |

### Backend
| Technology | Role |
|---|---|
| **Node.js v18+** | Runtime |
| **Express.js** | REST API server on port 3000 |
| **Socket.IO** | Real-time bidirectional events to frontend |
| **node-ssh** | SSH client — connects to Chatak nodes for config/control |
| **`net` (TCP)** | Raw TCP listeners that receive JSON streams from ODAS |
| **`child_process`** | Spawns Python scripts (bandpass filter, audio processing) |
| **`database_manager.js`** | File-based database (custom TXT format, no SQL) |
| **`src/scanner.js`** | Network scanner — discovers ODAS devices on local subnet |
| **`src/scanner-api.js`** | SSE (Server-Sent Events) API wrapping the scanner |

### Frontend
| Technology | Role |
|---|---|
| **React 18** | UI framework |
| **TypeScript** | Type safety across all components |
| **Vite** | Build tool and dev server (HMR) |
| **Tailwind CSS** | Utility-first styling |
| **shadcn/ui** | Accessible UI component library |
| **React Router (HashRouter)** | Client-side navigation |
| **TanStack Query (React Query)** | Server state management, caching, refetching |
| **Three.js** | spacital sound hemisphere rendering |
| **LightningChart JS (`@arction/lcjs`)** | High-performance real-time spectrograms |
| **Recharts** | Time-series charts, statistics |
| **Leaflet + leaflet.offline** | 2D map rendering with offline tile support |
| **Google Maps JS API** | Online map alternative |
| **Lucide Icons** | Icon system |

### Testing
| Technology | Role |
|---|---|
| **Vitest** | Backend unit tests and integration tests |
| **Playwright** | E2E browser automation tests |

### Build & Distribution
| Technology | Role |
|---|---|
| **electron-builder** | Packages app into Windows NSIS installer |
| **pkg** | Optional bundling of backend as standalone binary |

---

## 4. 📁 Project File Structure

```
SonicWild_Ship/
│
├── main.js                      # Electron entry point
├── preload.js                   # Electron preload (contextBridge)
├── server.js                    # ⭐ Main backend — Express + Socket.IO + TCP (108KB)
├── server- Base.js              # Older base version of server for reference
├── database_manager.js          # File-based database CRUD (11KB)
├── database.js                  # Alternate/migration DB helper
├── migrate.js                   # DB migration script
├── diagnose_rpi.js              # Raspberry Pi diagnostic tool
│
├── src/                         # Backend utilities
│   ├── scanner.js               # Network scanner (finds ODAS devices)
│   ├── scanner-api.js           # SSE API wrapping scanner
│   ├── interrogateDevice.js     # SSH interrogation helpers
│   ├── ping.js                  # Ping utility
│   ├── logger.js                # Centralized logging
│   ├── cache.json               # Local discovery cache
│   ├── ssh_success.json         # Cache of successful SSH IPs
│   └── ssh_failures.json        # Cache of failed SSH IPs
│
├── frontend/                    # ⭐ React + Vite frontend
│   ├── src/
│   │   ├── App.tsx              # Root — Providers + Router
│   │   ├── main.tsx             # React entry point
│   │   ├── index.css / App.css  # Global styles
│   │   ├── config/
│   │   │   └── environment.ts   # Reads SONICWILD_CONFIG (API/Socket URLs)
│   │   ├── pages/               # Route-level page components
│   │   │   ├── DeviceConnection.tsx    # Device discovery & management
│   │   │   ├── Setup.tsx               # Project setup wizard (4-step / 3-step)
│   │   │   ├── LiveMonitoring.tsx      # Live acoustic monitoring (main page)
│   │   │   ├── LiveVisualization.tsx   # spacital visualization view
│   │   │   ├── ArchiveViewer.tsx       # Archive browsing & playback
│   │   │   ├── Dashboard.tsx           # Overview dashboard
│   │   │   ├── ProjectDashboardPage.tsx
│   │   │   ├── ProjectListPage.tsx
│   │   │   ├── DeviceManagement.tsx
│   │   │   ├── DeviceSettingsPage.tsx
│   │   │   └── ManageClassifierPage.tsx
│   │   ├── components/          # Reusable UI components
│   │   │   ├── DevicePopup.tsx         # ⭐ Large per-device panel (79KB)
│   │   │   ├── spectrogram.tsx         # Spectrogram visualizer
│   │   │   ├── terrain/                # Google Maps terrain components
│   │   │   ├── three/                  # Three.js spacital hemisphere components
│   │   │   ├── visualization/          # Chart and plot components
│   │   │   ├── audio/                  # Audio player/streaming components
│   │   │   ├── archive/                # Archive browser UI
│   │   │   ├── device/                 # Device status/control panels
│   │   │   ├── deployment/             # Mic deployment UI
│   │   │   ├── setup/                  # Setup wizard steps
│   │   │   ├── project/                # Project management UI
│   │   │   ├── layout/                 # Sidebar, header, shell
│   │   │   ├── dashboard/              # Dashboard widgets
│   │   │   ├── gps/                    # GPS coordinate UI
│   │   │   └── ui/                     # shadcn/ui base components
│   │   ├── hooks/               # Custom React hooks
│   │   │   ├── useRealTimeData.ts
│   │   │   ├── useTriangulationData.ts
│   │   │   └── useDeviceManagement.ts
│   │   └── services/            # API and socket clients
│   │       ├── api.ts           # REST API calls
│   │       ├── socket.ts        # Main Socket.IO connection
│   │       └── visualizationSocket.ts
│   └── package.json             # Frontend dependencies
│
├── public/                      # Static assets served by Express
│   ├── frontend-dist/           # Built React app (for production)
│   ├── maps/                    # Offline Leaflet tile cache
│   └── js/, images/             # Additional static assets
│
├── tests/                       # Test suites
│   ├── backend/                 # Vitest unit tests
│   ├── integration/             # Integration tests
│   └── e2e/                     # Playwright browser tests
│
├── zodas_database.txt           # Live file-based database
├── zodas.db                     # SQLite DB (migration target)
├── package.json                 # Root package — Electron config, scripts
├── playwright.config.ts
├── vitest.config.backend.ts
├── vitest.config.integration.ts
├── start-sonicwild.bat          # Field-use launcher script
├── BUILD_SONICWILD.bat          # Full build script
└── README.md                    # High-level overview
```

---

## 5. 📱 Pages & Features (Screen-by-Screen)

### 5.1 Device Connection (`DeviceConnection.tsx`)
The **entry point** for every session. Researchers discover and connect to Chatak hardware here.

**Features:**
- **Auto-Discovery**: Triggers `src/scanner.js` via SSE endpoint (`GET /api/devices`) — scans the local subnet for devices running ODAS
- **Manual IP Entry**: Fallback for devices not found by scan
- **Device List**: Shows each discovered device with its IP, online/offline status, device ID, and SSH reachability
- **SSH Credentials**: Enter `userId` / `password` (default: `chatak`/`chatak`) per device
- **Device Interrogation**: Reads live status from `/home/chatak/ChatakGUI/config/` on the hardware — including `deviceId.txt`, `project_name.txt`, `record.txt`, `pause.txt`
- **Connection Persistence**: Successful IPs cached in `src/ssh_success.json`

### 5.2 Setup / Project Wizard (`Setup.tsx` — 106KB)
The **configuration hub** — where researchers define their deployment before going live.

**Live Session (4-Step Wizard):**
1. **Step 1 — Kit & Coverage**: Set WiFi hotspot location on map; define coverage range visually
2. **Step 2 — Mic Placement**: Drag Chatak nodes to their real physical positions on the map; set sensing range
3. **Step 3 — Session Config**: Choose target species (frogs, primates, birds); set recording mode (4ch / 1ch / none)
4. **Step 4 — Confirm & Launch**: Pre-flight checklist; sync settings to hardware

**Passive Session (3-Step Wizard):**
1. **Step 1 — Grid Setup**: Auto-calculate grid center; suggest mic ranges
2. **Step 2 — Deployment Schedule**: Set duty cycle (e.g., record 10 min/hour)
3. **Step 3 — Confirm**: Final check and deploy

**Key Data Synced:**
- Base Camp GPS coordinates (green marker)
- Microphone GPS positions
- Recording window (`recordMin`) and pause cycle (`pauseMin`)
- Project name and site description

### 5.3 Live Monitoring (`LiveMonitoring.tsx` — 65KB)
The **primary operational screen** used every day in the field.

**Features:**
- **2D Map View** (Google Maps + Leaflet offline):
  - **Blue Rays (Tracks)**: Stable, ODAS-confirmed sound sources with consistent acoustic signatures
  - **Red Rays (Pots)**: Short-lived potential detections / noise spikes
  - Each ray originates from its microphone and points in the direction of the sound source
- **spacital Hemisphere Viewer** (Three.js): Click any mic to open a "Sound Bubble" showing elevation (0°–90°) and azimuth (0°–360°) of incoming sounds
- **Spectrogram** (LightningChart JS): Real-time frequency visualization (0–20kHz) of the audio stream from any device
- **Live Audio Streaming**: Listen to mono or 4-channel audio in real-time from any connected Chatak node
- **Device Controls**: Start/stop recording, push config updates, view device telemetry
- **Triangulated Position**: Overlay showing the calculated spacital position of a dominant sound source computed from multiple sensors

### 5.4 Archive Viewer (`ArchiveViewer.tsx`)
For **post-deployment analysis** — reviewing what was recorded.

**Features:**
- Browse `.raw` and `.wav` files stored on connected Chatak nodes via SSH
- Stream or download archives
- Synchronized playback with spectrogram and spacital visualization
- Recent files list with quick-access

### 5.5 Dashboard (`Dashboard.tsx`)
**Overview screen** showing:
- Active projects summary
- Connected device count and statuses
- Recent activity log
- Quick navigation to key workflows

### 5.6 Project Management (`ProjectListPage.tsx`, `ProjectDashboardPage.tsx`)
- List and CRUD for research projects
- Each project stores: name, description, base GPS, linked mic locations, settings
- Per-project dashboard with stats

### 5.7 Device Management & Settings (`DeviceManagement.tsx`, `DeviceSettingsPage.tsx`)
- Manage per-device configurations
- Update SSH credentials, recording windows, network settings
- Device health monitoring

### 5.8 Classifier Management (`ManageClassifierPage.tsx`)
- Upload TFLite AI models (BirdNET, custom species classifiers) to Chatak nodes
- Manage model lifecycle (add, update, remove)
- Track model accuracy per project *(partially implemented)*

---

## 6. 🔄 Data Flow (End-to-End)

```
ODAS Hardware (Chatak Node)
        │
        │  TCP JSON Stream (60Hz, port varies per device)
        │  Format: { "src": [...tracks], "pot": [...pots] }
        ↓
server.js TCP Listener (per device, port 8081+)
        │
        │  Custom brace-counting JSON parser
        │  (fragments raw TCP byte stream → discrete JSON objects)
        ↓
Socket.IO Rooms
  ├── "track_info"          → raw tracks array
  ├── "triangulatedPosition" → computed spacital source location
  └── "pot_info"            → potential detections
        │
        ↓
React Frontend (Socket.IO client)
  ├── useRealTimeData hook → LiveMonitoring.tsx
  ├── useTriangulationData hook → LiveVisualization.tsx
  └── DevicePopup.tsx → Three.js + LightningChart rendering
        │
        ↓
Rendered in UI:
  ├── 2D Map: Directional rays from mic positions
  ├── spacital Sphere: Azimuth/Elevation hemisphere
  └── Spectrogram: Frequency vs time heatmap
```

### Device Discovery Flow
```
User clicks "Scan"
        ↓
GET /api/devices (SSE stream)
        ↓
src/scanner-api.js → spawns scanner.js
        ↓
scanner.js pings local subnet (e.g., 192.168.1.0/24)
        ↓
For each responding IP → SSH interrogation via interrogateDevice.js
        ↓
Results streamed back to frontend via SSE
        ↓
DeviceConnection.tsx renders device cards in real-time
```

### SSH Config Sync Flow
```
User selects device + enters credentials
        ↓
POST /sync-values { ip, userId, password }
        ↓
node-ssh connects to device
        ↓
Reads /home/chatak/ChatakGUI/config/:
  - deviceId.txt
  - project_name.txt
  - record.txt
  - pause.txt
        ↓
Returns settings object to frontend
        ↓
Frontend displays current device state
```

---

## 7. 🌐 REST API Reference (Backend — Port 3000)

### Device & SSH
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/sync-values` | SSH into a device and sync its current config |
| `GET` | `/global-settings` | Get active cached session settings |
| `POST` | `/updateSettings` | Push new settings to device via SSH |
| `POST` | `/run-command` | Execute a shell command on a device via SSH |
| `POST` | `/stop-command` | Stop a running command on a device via SSH |
| `GET` | `/api/devices` | SSE stream — real-time device discovery results |

### Projects
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/save-project` | Create or update a project |
| `GET` | `/projects` | List all projects |
| `GET` | `/project/:id` | Get a single project by ID |
| `DELETE` | `/project/:id` | Delete a project |

### Mic Locations
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/save-mic-location` | Save or update a mic's GPS position |
| `GET` | `/mic-locations` | List all saved mic locations |

### Device Configs
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/save-device-config` | Save IP + credentials for a device |

### Maps & Tiles
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/load-tile` | Serve an offline Leaflet map tile from local filesystem |
| `GET` | `/maps/manifest` | Get the manifest of available offline tile bundles |

### Archive
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/archive/list` | List archive files on a connected device (via SSH) |
| `GET` | `/archive/download` | Download an archive file from the device |

### Socket.IO Events (Real-Time)
| Event | Direction | Description |
|---|---|---|
| `track_info` | Server → Client | Array of current sound tracks from ODAS |
| `pot_info` | Server → Client | Array of potential detections |
| `triangulatedPosition` | Server → Client | Computed spacital source location `{x, y, z, lat, lng}` |
| `device_status` | Server → Client | Device online/offline change |

---

## 8. 🗄️ Database & Data Structures

### Primary Database: `zodas_database.txt`
Managed by `database_manager.js`. Stored in `%APPDATA%/SonicWild` (production) or project root (dev). Uses a custom bracket-tagged format:

```
[metadata]
{"version": "1.0", "schema": "sonicwild-v1"}

[projects]
[{"id":"1716900000000","name":"Dhurandhar Site","description":"...","baseLocation":{"lat":21.1,"lng":79.0},...}]

[micLocations]
[{"id":"mic_001","name":"Node A","lat":21.101,"lng":79.001,"online":true,"deviceId":"chatak-01","projectId":"1716900000000"}]

[deviceConfigs]
[{"ip":"192.168.1.101","userId":"chatak","password":"chatak","recordMin":"30","pauseMin":"5"}]
```

### Project Object
```typescript
{
  id: string;            // Unix timestamp string (e.g., "1716900000000")
  name: string;          // Human-readable project name
  description: string;   // Site context
  baseLocation: {
    lat: number;
    lng: number;
  };
  micLocations: string[]; // Array of mic IDs belonging to this project
  settings: object;       // Global config overrides
}
```

### Mic Location Object
```typescript
{
  id: string;            // e.g., "mic_001"
  name: string;          // Alias (e.g., "Node A - Riverbank")
  lat: number;
  lng: number;
  online: boolean;       // SSH reachability
  deviceId: string;      // Hardware ID from deviceId.txt
  projectId: string;     // Parent project ID
}
```

### Device Config Object
```typescript
{
  ip: string;            // e.g., "192.168.1.101"
  userId: string;        // SSH username (default: "chatak")
  password: string;      // SSH password (default: "chatak")
  recordMin: string;     // Recording window in minutes
  pauseMin: string;      // Duty cycle pause in minutes
}
```

### ODAS Track JSON (from hardware via TCP)
```typescript
{
  src: Array<{
    id: number;          // Track ID
    tag: string;         // "target" or "noise"
    x: number;           // Direction cosine X
    y: number;           // Direction cosine Y
    z: number;           // Direction cosine Z
    activity: number;    // Energy level 0.0–1.0
  }>;
  pot: Array<{           // Same structure as src
    ...
  }>;
}
```

### Runtime Cache Files
| File | Purpose |
|---|---|
| `src/cache.json` | Generic system cache for fast lookups |
| `src/ssh_success.json` | IPs that SSH'd successfully (speeds up re-discovery) |
| `src/ssh_failures.json` | IPs that failed SSH (skip in future scans) |

---

## 9. 🔌 Hardware: Chatak Nodes (ODAS Devices)

- **Hardware**: Raspberry Pi (any variant with 4-channel audio hat)
- **Software on device**: ODAS (Open embeddeD Audition System) — open-source spatial audio processing
- **SSH Access**: Username `chatak`, Password `chatak` (default)
- **Config directory on device**: `/home/chatak/ChatakGUI/config/`
  - `deviceId.txt` — unique device identifier
  - `project_name.txt` — current assigned project
  - `record.txt` — current recording state
  - `pause.txt` — duty cycle pause setting
- **TCP output**: ODAS emits two JSON streams:
  - **Tracks** (port 9000 default): Stable sound sources `src[]`
  - **Pots** (port 9001 default): Potential detections `pot[]`
- **Audio streaming**: Available via SSH tunnel or direct network stream

---

## 10. 🧪 Testing

### Backend Unit Tests
```bash
npm run test:backend
# Uses: Vitest, config: vitest.config.backend.ts
# Location: tests/backend/
```

### Integration Tests
```bash
npm run test:integration
# Uses: Vitest, config: vitest.config.integration.ts
# Location: tests/integration/
```

### E2E Tests (Playwright)
```bash
npm run test:e2e
# Uses: Playwright, config: playwright.config.ts
# Location: tests/e2e/
# Covers: Navigation, live monitoring, settings flows, archive viewer
```

### Frontend Tests
```bash
cd frontend && npm test
```

---

## 11. 🚀 Development Setup

### Prerequisites
- Windows 10/11 (primary target)
- Node.js v18+
- npm (bundled with Node)
- Google Maps API key (optional, for online maps)

### Run in Development Mode
```bash
# 1. Install root dependencies
npm install

# 2. Install frontend dependencies
cd frontend && npm install && cd ..

# 3. Start backend (port 3000)
node server.js

# 4. Start frontend dev server (port 8080)
cd frontend && npm run dev

# 5. Access in browser: http://localhost:8080
```

### Build for Production (Windows Installer)
```bash
# Full build (frontend → Electron package)
npm run package
# Output: dist/*.exe
```

### Key Ports
| Port | Service |
|---|---|
| `3000` | Express REST API + Socket.IO |
| `8080` | Vite dev server (React frontend) |
| `8081+` | Dynamic TCP listeners per Chatak device |
| `9090/9091` | Meta Fusion Server (C++ triangulation bridge) |

---

## 12. 🗺️ Git & Versioning

### Branch Strategy
- `release-v1.1-live-export` — **Current main development branch** (production-ready)
- Feature branches: `fix/*`, `feature/*`

### Key Tags (Restore Points)
| Tag | Description |
|---|---|
| `v1.1-working-may29` | ✅ Working snapshot May 29, 2026 — all features functional |
| `v1.1-stable` | Earlier stable baseline |

### How to Restore a Working Version
```bash
# View the working snapshot
git checkout v1.1-working-may29

# Create a new branch from the working snapshot to fix issues
git checkout -b fix/restore v1.1-working-may29

# Hard reset to the working state (⚠️ drops newer changes)
git reset --hard v1.1-working-may29
```

---

## 13. 🔮 Roadmap & Future Features

### ✅ Currently Implemented
- Auto-discovery of ODAS devices via SSE + subnet scanning
- SSH integration (config sync, remote command execution)
- Real-time TCP stream ingestion (60Hz) with custom JSON parser
- spacital triangulation visualization (Three.js hemispheres)
- Live spectrogram (LightningChart JS)
- 2D map with directional rays (Google Maps + Leaflet offline)
- Project and mic location CRUD (file-based DB)
- Archive browser and viewer (SSH file listing)
- Electron desktop distribution (Windows installer)
- E2E test suite (Playwright)
- Live Session Wizard (4-step) + Passive Session Wizard (3-step)

### 🔄 In Progress / Planned
- **TFLite Integration**: Upload and deploy AI models (BirdNET, etc.) to Chatak nodes directly from the UI
- **Classifier Management**: Track model versions and accuracy per project
- **Bulk Device Operations**: Push settings to multiple devices simultaneously
- **Connection Persistence**: Auto-reconnect on intermittent field WiFi
- **Biodiversity Reports**: One-click generation of activity reports from acoustic metadata
- **Internal Audio Import**: Search and retrieve audio by date range and mic ID
- **Professional Website**: Company-facing marketing website for deployment on company domain (planned)

---

## 14. 🌍 Professional Website Plan

The team is planning a **professional public-facing website** to position SonicWild alongside competitors like Synature. This website will:

- **Target audience**: Conservation researchers, wildlife NGOs, universities, government agencies
- **Key sections**:
  - Hero: "See where nature speaks" — spacital visualization demo
  - Product features (real-time spacital, offline, multi-sensor)
  - Use cases (tropical forests, wetlands, urban soundscapes)
  - How it works (architecture diagram, workflow)
  - Team / Company
  - Contact / Request Demo
- **Tech stack for website**: To be determined (Next.js or static HTML for CDN hosting)
- **Deployment**: Company domain (e.g., `sonicwild.io` or similar)
- **Inspiration**: Synature.com — clean, dark, nature-themed, professional SaaS aesthetic

---

## 15. ❓ FAQ for AI Assistants

**Q: What is a "Track" vs a "Pot"?**
> A: **Track** (`src[]`) — a stable, confirmed sound source identified by ODAS with consistent acoustic signature. Shown as **Blue rays** on the map. **Pot** (`pot[]`) — a short-lived potential detection or noise spike. Shown as **Red rays**.

**Q: Why does the map show blank/tiles missing?**
> A: Online mode needs a valid Google Maps API key (set in `.env`). Offline mode needs Leaflet tiles cached in `public/maps/`. Use the tile download feature in Settings.

**Q: How does SonicWild know where a sound is in spacital space?**
> A: ODAS runs on each Chatak node and performs beamforming + source localization using the 4-channel microphone array. It outputs direction cosines (x, y, z) per sound source. SonicWild converts these to Azimuth/Elevation for the spacital hemisphere and projects them onto the 2D map using the mic's GPS coordinates.

**Q: Can it work without internet?**
> A: Yes. SonicWild is fully offline-capable. The backend runs locally, maps are cached, and ODAS devices connect via local WiFi/LAN.

**Q: What is the default SSH login for Chatak nodes?**
> A: Username: `chatak`, Password: `chatak`

**Q: Where is the database stored?**
> A: Development: project root as `zodas_database.txt`. Production: `%APPDATA%/SonicWild/zodas_database.txt`

**Q: What is "Meta Fusion Server"?**
> A: A local process (C++ binary or wrapper) running on ports 9090/9091 that handles multi-sensor triangulation — combining angle data from multiple Chatak nodes to compute a single spacital source position. Results are fed into Socket.IO as `triangulatedPosition` events.

---

*Document maintained by the SonicWild engineering team.*
*Last updated: July 17, 2026 — v1.2 refresh.*

---

## 16. 2026 Refresh Notes (Current Code Reality)

- Backend runtime remains `server.js` (Express + Socket.IO), with SSH orchestration against Chatak nodes.
- Frontend runtime served by backend is built output in `public/frontend-dist`; after frontend edits, run `npm run build:frontend`.
- Lat/long behavior is tied to `/home/chatak/ChatakGUI/config/latlong.txt`, surfaced through `/sync-values`.
- Update settings now supports sentinel-style optional arguments (for preserving existing device values).
- Bulk/per-device settings UX has evolved significantly from early snapshots, including config-file selection workflows.

---

## 17. Month-over-Month Changes (From Git History)

### 2026-07
- Reliability and UX fixes in live monitoring/device popup flows.
- Update Settings and config-file handling refinements.
- Stop/close process controls tuned to avoid killing unrelated long-running ODAS jobs.
- Documentation updates and cleanup commits.
- Includes frequent `sprint-hourly` automated checkpoint commits.

### 2026-05
- RAG and file manager updates.
- Safe local audio/JSON improvements.

### 2026-04
- Large wave of feature delivery: bulk device updates, map/location integration, spectrogram and hemisphere improvements.
- Database path evolution and SQLite migration work.

### 2026-03
- Dashboard/UI polish and roadmap docs.

### 2026-02
- Initial stable baseline and project-level documentation updates.

### Maintenance rule
- Rebuild this section monthly from:
  `git log --date=format:%Y-%m --pretty=format:"%ad|%h|%s"`
