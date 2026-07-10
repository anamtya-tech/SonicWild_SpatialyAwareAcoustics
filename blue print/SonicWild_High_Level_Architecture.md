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
```

### 2) Software Responsibility Split (Edge Side vs Device Side)

```mermaid
flowchart LR
    subgraph EDGE_SIDE[Edge Side on Raspberry Pi]
        ODAS[Z_ODAS / ODAS\nC Direction of Arrival\nAudio Processing]
        PY[Python Orchestration\nSettings Update\nAudio Stream Management\nFrontend Request Handling\nDrive Mount and Management]
        AI[TFLite / YAMNet Integration\nInference in C/Python Pipeline]
        ODAS --> PY --> AI
    end

    subgraph DEVICE_SIDE[Device Side UI]
        FE[Node.js + React Frontend\nLive Visualization\nFile Visualization\nDevice Management]
    end

    EDGE_SIDE -->|Socket/HTTP Data + Control| DEVICE_SIDE
```

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