# SonicWild AI Knowledge Base (RAG Source)
**Project: SonicWild-EcoAcoustics**
**Version: 1.0.0**
**Target: Grok AI / RAG System Integration**

---

## 1. Project Mission & Identity
**SonicWild** is a high-performance, spatial-aware ecoacoustic monitoring platform designed for wilderness, forestry, and urban research. Its core mission is to provide researchers with a **spacital window into the acoustic environment**, allowing them to not just hear sounds, but to see exactly where they originate in space (Azimuth, Elevation, and Distance).

### One-Sentence Pitch:
"SonicWild is an Electron-based desktop ecosystem for discovering, managing, and visualizing real-time spacital sound triangulation and bioacoustic data from remote field sensors."

---

## 2. System Architecture (The "Brain")
SonicWild follows a distributed, triple-layer architecture:

### A. Physical Layer (Hardware)
- **Chatak Nodes**: Custom Raspberry Pi-based sensors running **ODAS** (Open embeddeD Audition System).
- **Function**: Capture raw 4-channel audio, perform local DSP (Digital Signal Processing), and emit tracking data (JSON) via TCP.

### B. Backend Layer (Control Center)
- **Engine**: Node.js & Express.
- **Real-time Gateway**: Socket.IO for duplex communication with the frontend.
- **Data Ingestion**: Custom TCP listeners that parse high-frequency JSON streams from hardware at 60Hz.
- **Bridge**: `node-ssh` for remote command execution (start/stop recording, update configs) and status retrieval.
- **Database**: `database_manager.js` (Portable, file-based TXT database using a custom tagged format).

### C. Frontend Layer (User Interface)
- **Framework**: React 18 with Vite and TypeScript.
- **Styling**: Tailwind CSS + shadcn/ui.
- **Visualization**: 
  - **Three.js**: Renders spacital sound hemispheres.
  - **LightningChart JS / Recharts**: High-speed spectrograms and time-series plots.
  - **Leaflet / Google Maps**: 2D spatial context for device deployment.

---

## 3. Core Features & Capabilities

### 📡 Device Management
- **Auto-Discovery**: Scans local network for ODAS devices using SSE (Server-Sent Events).
- **SSH Control**: Securely execute shell commands on sensors to sync clocks, start/stop recording, or reboot.
- **Interrogation**: Reads device status (Battery, Memory, Signal, Active Config) directly from the filesystem over SSH.

### 🔊 Live Monitoring
- **spacital Triangulation**: Projects sound sources as Blue Rays (Tracks - consistent) and Red Rays (Pots - potential) on a map.
- **spacital Hemisphere**: A "Sound Bubble" view that shows the elevation and azimuth of sounds relative to the microphone.
- **Spectrograms**: Real-time visual frequency analysis of the audio stream.
- **Live Audio**: Remote streaming of mono or 4-channel audio for real-time listening.

### 🗺️ Spatial Awareness
- **Base Camp Calibration**: Set a reference GPS coordinate for the entire deployment.
- **Mic Placement**: Drag-and-drop microphones on a map to define their real-world spatial coordinates.
- **Offline Maps**: Support for Leaflet tiles stored locally for deployments in remote areas without internet.

### 📂 Data & Archives
- **Project Organization**: Group deployments into "Projects" with shared metadata and base camps.
- **Archive Viewer**: Browse `.raw` or `.wav` files stored on remote sensors and play them back with full visualization sync.
- **Database**: Stores all settings in `zodas_database.txt`, making the entire app "portable" and easy to back up.

---

## 4. Technical Data Structures (For Deep Querying)

### Project Object
```json
{
  "id": "String (Timestamp)",
  "name": "Project Name",
  "description": "Context about the site",
  "baseLocation": { "lat": Float, "lng": Float },
  "micLocations": ["Array of Mic IDs"],
  "settings": { "Global Configs" }
}
```

### Mic Location Object
```json
{
  "id": "String",
  "name": "Mic Alias",
  "lat": Float, "lng": Float,
  "online": Boolean,
  "deviceId": "HW-ID",
  "projectId": "Parent-Project-ID"
}
```

### Device Configuration Object
```json
{
  "ip": "Local IP",
  "userId": "Default: chatak",
  "password": "Default: chatak",
  "recordMin": "Active recording window",
  "pauseMin": "Duty cycle pause"
}
```

---

## 5. User Workflows (How-To)

### Setup a New Site:
1. Open **Device Connection** -> Scan/Find your Chatak nodes.
2. Go to **Setup** -> Create a new Project.
3. Set **Base Camp** (Green Marker) to the center of your site.
4. Drag your **Microphones** to their physical locations.
5. Hit **Sync** to push coordinates to the hardware.

### Start Live Monitoring:
1. Navigate to **Live Monitoring**.
2. Select a project from the dropdown.
3. Observe the "Rays" on the map.
4. Click a Microphone to open the **spacital Sphere Viewer** for elevation data.

### Retrieve Data:
1. Go to **Archive Viewer**.
2. Select a connected device.
3. Browse files -> Download or Stream for playback.

---

## 6. FAQ & Troubleshooting (For AI Responses)

**Q: Why is my map blank?**
A: Ensure internet access for Google Maps. If offline, verify that the `maps/` directory contains downloaded Leaflet tiles for your current zoom level.

**Q: What is the difference between Blue and Red rays?**
A: **Blue (Tracks)** are stable sound sources identified by the ODAS algorithm. **Red (Pots)** are short-lived potential sound detections or noise spikes.

**Q: How do I stop the application?**
A: Go to the black terminal window (SonicWild Launcher) and press any key. It will gracefully kill all background Node processes and the TCP listeners.

**Q: Can I use SonicWild without internet?**
A: Yes! It is designed for field use. It uses a local Express server and offline Leaflet tiles.

**Q: What port does the backend use?**
A: The main API runs on port `3000`. The frontend runs on `8080`.

---

## 7. Future Roadmap (Upcoming Intelligence)
- **TFLite Integration**: Deploying AI classifiers (BirdNET, etc.) directly to the Chatak nodes.
- **Passive Monitoring Scheduler**: Setting complex duty cycles (e.g., record for 10 mins every hour).
- **Biodiversity Reporting**: One-click generation of activity reports from acoustic metadata.

---
**Confidentiality Note:** This document is for internal use for training the Grok AI chatbot to support SonicWild users.

---

## 8. 2026 Current-State Refresh (For RAG Accuracy)

- Canonical backend is `server.js` on port 3000 with Socket.IO and SSH control flows.
- Frontend source changes require rebuild to `public/frontend-dist` (`npm run build:frontend`) for runtime parity.
- Device lat/long source path in active flows is `/home/chatak/ChatakGUI/config/latlong.txt`.
- `sync-values` remains the key API for reconciling device-side settings into frontend state.
- Update Settings now supports sentinel-style optional values to preserve existing device fields when needed.

---

## 9. Month-over-Month (MoM) Change Tracking

### 2026-07
- Live monitoring/device popup stability and stop-flow behavior fixes.
- Config selection and update-settings behavior refinements (per-device + bulk workflows).
- Documentation refresh and cleanup commits.
- Month includes high-frequency `sprint-hourly` automation commits.

### 2026-05
- RAG-oriented updates and file manager changes.

### 2026-04
- Significant feature buildout across UI, mapping, bulk operations, spectrogram/visualization, and DB migration direction.

### 2026-03
- Dashboard/layout improvements and roadmap-related documentation work.

### 2026-02
- Stable baseline and initial project documentation updates.

### Monthly update method
- Use: `git log --date=format:%Y-%m --pretty=format:"%ad|%h|%s"`
- Summarize by month; separate automation/checkpoint commits from functional changes.
