# SonicWild Spatially Aware Acoustics

## High-Level Architecture

This document reformats the high-level architecture into a professional, implementation-oriented view.

### 1) System Line Diagram (Hardware + Software Placement)

```mermaid
flowchart TB
    subgraph EDGE[EDGE]
        direction TB

        subgraph HW[HW]
            direction LR
            MIC((ReSpeaker USB 4-Mic Array)) --> CABLE[Micro USB -> USB Cable]
            CABLE --> RPI[Raspberry Pi 3B]
            RPI --> PWR[Power Bank]
        end

        subgraph SW[SW]
            direction LR
            SW1[1. Z_ODAS: C code for direction calculation and audio processing]
            SW2[2. Python: settings update, audio streaming, frontend requests, drive management]
            SW3[3. TFLite: models and libraries for C integration]
        end
    end

    style EDGE fill:#efe7c7,stroke:#111827,stroke-width:2px,color:#ffffff
    style HW fill:#cfe8e3,stroke:#22b8a8,stroke-width:1.5px,color:#111111
    style SW fill:#dbe0ef,stroke:#6b7aa8,stroke-width:1.5px,color:#111111
    style MIC fill:#69b3f2,stroke:#1c6fb8,stroke-width:2px,color:#111111
    style CABLE fill:#495057,stroke:#212529,stroke-width:1.5px,color:#ffffff
    style RPI fill:#b2f2bb,stroke:#2b8a3e,stroke-width:1.5px,color:#111111
    style PWR fill:#ffd43b,stroke:#e67700,stroke-width:1.5px,color:#111111
    style SW1 fill:#b7d4f3,stroke:#4f8fca,stroke-width:1.5px,color:#111111
    style SW2 fill:#b7d4f3,stroke:#4f8fca,stroke-width:1.5px,color:#111111
    style SW3 fill:#b7d4f3,stroke:#4f8fca,stroke-width:1.5px,color:#111111
```

### 2) Software Responsibility Split (Edge Side vs Device Side)

```mermaid
flowchart TB
    subgraph TOP_ROW[ ]
        direction LR
        PHONE[Mobile Hotspot Bridge<br/>No Internet Required]
        LAPTOP[Laptop Bridge Device<br/>No Internet Required]
        REC[Recorded Audio<br/>Input Stream]
    end

    subgraph BOTTOM_ROW[ ]
        direction LR
        VIEW[Any Device for Viewing<br/>No Internet Required]
        FE[On-Device Frontend<br/>Visualizer + Device Manager<br/>Node.js + React]
    end

    PHONE --> VIEW
    LAPTOP --> VIEW
    REC --> VIEW
    VIEW --> FE

    style PHONE fill:#ffadad,stroke:#c92a2a,stroke-width:2px,color:#111111
    style LAPTOP fill:#caffbf,stroke:#2f9e44,stroke-width:2px,color:#111111
    style REC fill:#4c74c9,stroke:#1d3557,stroke-width:2px,color:#ffffff
    style VIEW fill:#fff3bf,stroke:#f08c00,stroke-width:2px,color:#111111
    style FE fill:#bde0fe,stroke:#1971c2,stroke-width:2px,color:#111111

    style TOP_ROW fill:#ffffff,stroke:#ffffff,stroke-width:0px,color:#111111
    style BOTTOM_ROW fill:#ffffff,stroke:#ffffff,stroke-width:0px,color:#111111
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