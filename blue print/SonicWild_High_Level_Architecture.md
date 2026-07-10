# SonicWild Spatially Aware Acoustics

## High-Level Architecture

This document reformats the high-level architecture into a professional, implementation-oriented view.

### 1) ASCII Block Diagram

```text
+------------------------------------------------------------------------------------+
|                                        EDGE                                        |
|                                                                                    |
|  +--------------------------------------- HW ------------------------------------+  |
|  |                                                                            |   |
|  |  (ReSpeaker USB 4-Mic Array) --[Micro USB -> USB Cable]--> [Raspberry Pi 3B] |
|  |                                                             --> [Power Bank]   |
|  |                                                                            |   |
|  +----------------------------------------------------------------------------+   |
|                                                                                    |
|  +--------------------------------------- SW ------------------------------------+ |
|  | [1. Z_ODAS: C code for direction calculation and audio processing]           | |
|  | [2. Python: settings update, audio streaming, frontend requests, drive mgmt] | |
|  | [3. TFLite: models and libraries for C integration]                          | |
|  +-------------------------------------------------------------------------------+ |
+------------------------------------------------------------------------------------+
```

## Repository Links By Section

### Edge Processing (ODAS)
- Repository: [SonicWild_ODAS_Edge](https://github.com/anamtya-tech/SonicWild_ODAS_Edge)
- README: [SonicWild_ODAS_Edge README](https://github.com/anamtya-tech/SonicWild_ODAS_Edge/blob/main/README.md)

### Raspberry Pi Setup and System Bring-Up
- Repository: [SonicWild_RPi_Setup](https://github.com/anamtya-tech/SonicWild_RPi_Setup)
- Documentation Entry: [SonicWild_RPi_Setup Architecture Notes](https://github.com/anamtya-tech/SonicWild_RPi_Setup/blob/main/PI_TECH_STACK_ARCHITECTURE.md)

### Frontend and Visualization Context
- Current architecture workspace: [SonicWild_SpatialyAwareAcoustics](https://github.com/anamtya-tech/SonicWild_SpatialyAwareAcoustics)
- Source slide used for this rewrite: [blue print/Sonicwild Block Diagram.pptx](https://github.com/anamtya-tech/SonicWild_SpatialyAwareAcoustics/blob/main/blue%20print/Sonicwild%20Block%20Diagram.pptx)

### Ongoing Research and Validation
- Repository: [SonicWild_Yammnet](https://github.com/anamtya-tech/SonicWild_Yammnet)
- Repository: [SonicWild_Simulator](https://github.com/anamtya-tech/SonicWild_Simulator)

## Section Explanations

### Hardware Layer
- ReSpeaker USB Mic Array captures multichannel audio.
- Raspberry Pi 3B is the edge compute node.
- Power Bank provides portable, field-ready power.

### Edge Side Software
- Z_ODAS / ODAS performs localization and core audio direction processing.
- Python services orchestrate runtime settings, manage audio stream flow, and bridge requests from UI/control clients.
- TFLite and YAMNet components are used for embedded acoustic event inference and ongoing model integration.

### Device Side Software
- Node.js + React frontend is used for visualization and device control.
- Supports live and recorded views and configuration workflows.

### Connectivity and Access
- Architecture supports local/offline operation through hotspot/laptop bridging.
- Viewer devices can access the system without requiring public internet.

## Repository Mapping

### Core Edge Repositories
- SonicWild_RPi_Setup: Raspberry Pi setup, deployment baseline, and system-level scripts.
- SonicWild_ODAS_Edge: edge ODAS/audio processing implementation.

### Ongoing Work and Research
- SonicWild_Yammnet: YAMNet experimentation and integration work.
- SonicWild_Simulator: simulation and analysis workflows.

## Notes
- This representation is derived from the current high-level architecture slide and your repo context.
- As implementation evolves, expand this doc with interface contracts, message schemas, and deployment topology.