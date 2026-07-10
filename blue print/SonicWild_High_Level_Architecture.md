# SonicWild Spatially Aware Acoustics

## High-Level Architecture

This document reformats the high-level architecture into a professional, implementation-oriented view.

### 1) System Line Diagram (Hardware + Software Placement)

```mermaid
flowchart TB
    subgraph TOP[Device]
        direction LR

        subgraph HW[HW]
            direction LR
            MIC((ReSpeaker USB 4-Mic Array)) -->|Micro USB -> USB Cable| RPI[Raspberry Pi 3B]
            RPI --> PWR[Power Bank]
        end

        subgraph SW[SW]
            direction TB
            SW1[1. Z_ODAS: C code for direction calculation and audio processing]
            SW2[2. Python: settings update, audio streaming, frontend requests, drive management]
            SW3[3. TFLite: models and libraries for C integration]
            SW4[4. Frontend visualizer and device manager in Node.js + React]
        end
    end

    subgraph USER[User]
        direction LR
        PHONE[Mobile hotspot bridge\nNo internet required]
        OR[OR]
        LAPTOP[Any laptop as bridge and for viewing\nNo internet required]
        VIEW[Any device for viewing\nNo internet required]
    end

    RPI -. local bridge .-> PHONE
    RPI -. local bridge .-> LAPTOP
    PHONE -. local access .-> VIEW
    LAPTOP -. local access .-> VIEW

    style MIC fill:#6ea8fe,stroke:#1d4ed8,stroke-width:2px,color:#111111
    style RPI fill:#8fd694,stroke:#2b8a3e,stroke-width:2px,color:#111111
    style PWR fill:#ffd43b,stroke:#e67700,stroke-width:2px,color:#111111
    style SW1 fill:#d0ebff,stroke:#1971c2,stroke-width:1.5px,color:#111111
    style SW2 fill:#d0ebff,stroke:#1971c2,stroke-width:1.5px,color:#111111
    style SW3 fill:#d0ebff,stroke:#1971c2,stroke-width:1.5px,color:#111111
    style SW4 fill:#d0ebff,stroke:#1971c2,stroke-width:1.5px,color:#111111
    style PHONE fill:#ffadad,stroke:#c92a2a,stroke-width:2px,color:#111111
    style LAPTOP fill:#caffbf,stroke:#2f9e44,stroke-width:2px,color:#111111
    style VIEW fill:#fff3bf,stroke:#f08c00,stroke-width:2px,color:#111111
    style OR fill:#ffffff,stroke:#495057,stroke-width:1.5px,color:#111111
```

### 2) Software Responsibility Split (Edge Side vs Device Side)

```mermaid
flowchart LR
    subgraph EDGE_SIDE[Edge Side on Raspberry Pi]
        subgraph ZODAS[Z_ODAS Processing Zone (~30% of Edge Software Area)]
            ODAS_CORE[ODAS Core in C\nDirection of Arrival\nParticle/Localization Stream]
            YAMNET_BOX[YAMNet/TFLite Module\n(~10% sub-area inside Z_ODAS)]
            ODAS_CORE <--> YAMNET_BOX
        end

        PY[Python Control and Streaming Layer\nSettings Update\nAudio Stream Management\nFrontend Request Handling\nDrive Mount and Management]

        ODAS_CORE <--> PY
        YAMNET_BOX <--> PY
    end

    subgraph DEVICE_SIDE[Device Side UI]
        FE[Node.js + React Frontend\nLive Visualization\nFile Visualization\nDevice Management]
    end

    ODAS_CORE -->|Particle Data: timestamp, id, type, x/y/z, confidence| FE
    PY <-->|Control Messages + Data Messages + Audio Streams| FE

    style ZODAS fill:#fff3bf,stroke:#f59f00,stroke-width:2px,color:#111111
    style ODAS_CORE fill:#ffd166,stroke:#e67700,stroke-width:3px,color:#111111
    style YAMNET_BOX fill:#ff99c8,stroke:#c9184a,stroke-width:2px,color:#111111
    style PY fill:#9bf6ff,stroke:#0077b6,stroke-width:2px,color:#111111
    style FE fill:#a0c4ff,stroke:#3a0ca3,stroke-width:2px,color:#111111
    style EDGE_SIDE fill:#f8f9fa,stroke:#495057,stroke-width:1px,color:#111111
    style DEVICE_SIDE fill:#f8f9fa,stroke:#495057,stroke-width:1px,color:#111111
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