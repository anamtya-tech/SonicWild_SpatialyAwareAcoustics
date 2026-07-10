# SonicWild Spatially Aware Acoustics

## High-Level Architecture

This document reformats the high-level architecture into a professional, implementation-oriented view.

### 1) System Line Diagram (Hardware + Software Placement)

```mermaid
flowchart LR
    subgraph HW[Hardware Layer]
        MIC[ReSpeaker USB Mic Array]
        RPI[Raspberry Pi 3B]
        PWR[Power Bank]
        MIC -->|USB Audio| RPI
        RPI --- PWR
    end

    subgraph ACCESS[User Access Layer]
        PHONE[Mobile Hotspot Bridge\nNo Internet Required]
        LAPTOP[Any Laptop as Bridge\nNo Internet Required]
        VIEWER[Any Device for Viewing\nNo Internet Required]
    end

    subgraph SW[Software Layer]
        EDGE[Edge Side\nSignal Processing + Control + AI]
        DEVICE[Device Side\nFrontend Visualizer + Device Manager]
    end

    RPI --> EDGE
    EDGE --> DEVICE
    PHONE -. local bridge .- DEVICE
    LAPTOP -. local bridge .- DEVICE
    VIEWER -. local access .- DEVICE

    style MIC fill:#7bdff2,stroke:#005f73,stroke-width:2px,color:#111111
    style RPI fill:#b8f2a5,stroke:#2b9348,stroke-width:2px,color:#111111
    style PWR fill:#ffd166,stroke:#e09f00,stroke-width:2px,color:#111111
    style EDGE fill:#ffc6ff,stroke:#b5179e,stroke-width:2px,color:#111111
    style DEVICE fill:#a0c4ff,stroke:#1d4ed8,stroke-width:2px,color:#111111
    style PHONE fill:#ffadad,stroke:#c1121f,stroke-width:2px,color:#111111
    style LAPTOP fill:#caffbf,stroke:#2d6a4f,stroke-width:2px,color:#111111
    style VIEWER fill:#fdffb6,stroke:#8f6a00,stroke-width:2px,color:#111111
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