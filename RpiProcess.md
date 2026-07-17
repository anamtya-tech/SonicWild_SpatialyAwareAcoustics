# SonicWild Processes Reference

This document summarizes Python scripts in `ChatakGUI`, `audio_stream`, and the CORS server deployed to `/mnt/CHATAK_VM/cors_http_server.py`.

## ChatakGUI (Active)

- `ChatakGUI/archive_process.py`: Builds archive workflows for ODAS outputs by creating directories, converting RAW to WAV, rewriting ODAS config paths, executing ODAS core, and compressing generated result folders for transfer.
- `ChatakGUI/asound_check.py`: Detects the ReSpeaker sound card dynamically, updates ALSA configuration safely, and logs configuration actions with timestamps so capture routes remain valid after reboots or hardware changes.
- `ChatakGUI/odas_manager.py`: Orchestrates ODAS execution lifecycle, config rewrites, bandpass reset, arecord startup, output monitoring, and graceful signal handling for synchronized localization and capture operations on the edge device.
- `ChatakGUI/record_odas.py`: Headless recording runner and also launched ODAS that verifies mount readiness, tracks recording status, launches arecord with correct parameters and saves the odas output for offline reporting, and logs operational events for unattended field recording sessions.
- `ChatakGUI/sys_check.py`: Polls system Wi-Fi status, interprets signal quality from command output, and provides normalized connectivity indicators used by monitoring or setup scripts during deployment validation.
- `ChatakGUI/update_bandpass.py`: CLI helper that updates bandpass-related configuration values from arguments, enabling runtime filter parameter changes without manually editing config files on the Raspberry Pi.
- `ChatakGUI/update_filePath.py`: Updates configuration file path references programmatically, simplifying environment migration and ensuring downstream processes read correct input and output locations after deployment changes.
- `ChatakGUI/update_ip.py`: Finds and replaces IP values in target files using regex-based edits, supporting quick endpoint retargeting when network topology or service addresses change.
- `ChatakGUI/update_settings.py`: Generic settings updater that writes key configuration values to target files from command-line inputs, reducing repetitive manual edits and improving repeatable provisioning steps.
- `ChatakGUI/usb_mount.py`: Executes and logs USB or remote mount commands, validating mount outcomes and recording timestamped diagnostics so storage dependencies are available before recording and archival jobs start.
- `ChatakGUI/wifi_connect.py`: Performs targeted Wi-Fi switching, retrieves current SSID, connects to selected network profiles, and optionally restores original connectivity after temporary network operations finish.
- `ChatakGUI/wifi_deamon.py`: Background Wi-Fi watchdog that scans networks, ensures WPA profiles exist, attempts reconnection, and logs connection state continuously for resilient remote-device field behavior.

## Reboot Cronjobs (Active)

- `ChatakGUI/Old/asound_check.py`: Earlier ALSA card-detection and asound.conf rewrite utility retained for rollback, preserving prior logic used before current logging and robustness improvements.

- `ChatakGUI/Old/usb_mount.py`: Earlier mount utility revision with timestamped command logging, retained for historical comparison and quick rollback if newer storage handling misbehaves.


## audio_stream (Active)

- `audio_stream/chatak_audio_stream.py`: Primary live stream server for PCM and WAV output, including port cleanup and arecord integration to serve near-real-time audio from edge capture devices.
- `audio_stream/file_chatak_audio_stream.py`: File-backed streaming variant that serves buffered audio using temporary files, useful for replay and controlled output scenarios beyond direct microphone pipelines.


## CORS HTTP Server (deployed on `/mnt/CHATAK_VM`)

- `cors_http_server.py`: Lightweight static HTTP server with CORS-enabled request handler, custom content-type behavior, and path translation logic for serving shared assets from mounted storage to frontend clients.


## #####################################################################################
    Below code is not Active but may be in place for backward compatibility
## #####################################################################################

## ChatakGUI/Old (Legacy)


- `ChatakGUI/my_script.py`: Main Tkinter control panel for operators, handling recording start and stop, process resume flow, parameter updates, and UI-triggered device-management tasks in one interactive desktop interface.
- `ChatakGUI/Recording.py`: Lightweight Tkinter recording utility that checks process state and provides simple start and stop controls for manual audio capture testing or operator-driven recording workflows.
- `ChatakGUI/Old/my_script (copy 1).py`: Legacy GUI variant supporting image-based navigation, live visualization windows, and IP update actions that informed later consolidated dashboard implementations.
- `ChatakGUI/Old/my_script0.py`: Older multi-window GUI prototype with image resizing, interim views, transfer operations, and dialog-driven interactions used during early operator workflow design.
- `ChatakGUI/Old/my_script030225.py`: Transitional control GUI adding pause and resume behavior, recording management, and updated process flow before features were merged into newer script revisions.
- `ChatakGUI/Old/my_script1.py`: Historic GUI implementation for transfer, settings updates, project metadata editing, and remote process interaction during initial end-user tool iteration.
- `ChatakGUI/Old/record.py`: Previous headless recording script version, including mount checks and arecord orchestration, kept as fallback reference during production script evolution.
- `ChatakGUI/Old/Recording.py`: Legacy basic recorder UI with process detection and record toggles, superseded by newer interfaces but preserved for compatibility and regression troubleshooting.
- `ChatakGUI/Old/temp.py`: Temporary helper containing repeated UI update routines for pause, record, wake, sleep, and device identifiers during exploratory GUI state-management development.
- `ChatakGUI/test.py`: Placeholder or sandbox script reserved for quick local experiments, ad hoc checks, and temporary test routines without affecting production capture or processing pipelines.


## audio_stream (Old for backeard compatibility)

- `audio_stream/audio_archive.py`: Flask-based audio streaming service that emits raw and WAV streams, includes shutdown controls, and supports archival-oriented stream handling for remote monitoring endpoints.
- `audio_stream/odas_archive.py`: Converts ODAS JSON-like outputs into frame-indexed track and potential-source structures, then emits formatted frame data over sockets for analytics or visualization consumers.
- `audio_stream/play_magic.py`: Main orchestrator for synchronized magic pipeline execution, wiring config parsing, buffering, ODAS progress wrappers, signals, and threaded lifecycle management into one command-line entrypoint.
- `audio_stream/play_magic_copy.py`: Alternate copy of play_magic flow used for experiments or fallback runs, preserving similar orchestration behavior while testing incremental pipeline adjustments.

## audio_stream/magic (Old tesing code)

- `audio_stream/magic/__init__.py`: Package entrypoint exposing CLI bootstrap behavior, argument parsing, and config initialization hooks for launching modular magic audio-processing components.
- `audio_stream/magic/1socket.py`: Minimal socket client abstraction for connecting and pushing JSON-line payloads to downstream consumers in early or simplified transport scenarios.
- `audio_stream/magic/buffer_audio.py`: Buffers audio chunks into temporary structures using config-driven policies, preparing synchronized audio segments for coordinated emission with ODAS-derived events.
- `audio_stream/magic/buffer_odas.py`: Parses ODAS progress output, runs wrapper processes, and launches threaded monitors that maintain synchronized localization data buffering for the magic pipeline.
- `audio_stream/magic/config.py`: Reads and parses configuration text files, extracts key numeric parameters, and centralizes config interpretation needed by stream, buffer, and orchestration modules.
- `audio_stream/magic/emitter.py`: Converts parsed ODAS objects into frame buckets and emits formatted per-frame events over sockets for real-time visualization, analytics, or synchronized playback alignment.
- `audio_stream/magic/emitter_copy.py`: Backup emitter implementation mirroring core formatting and socket emission behavior, retained for testing parser changes without risking main emitter stability.
- `audio_stream/magic/socket_client.py`: Reusable socket client class handling connect, close, and JSON-line transmission for multiple modules needing consistent outbound transport behavior.
- `audio_stream/magic/streams.py`: Flask streaming module generating raw and WAV outputs from chunk queues, enabling synchronized HTTP streaming interfaces that pair with ODAS-driven event emitters.
- `audio_stream/magic/sync_stream.py`: Coordinates synchronized startup timing between PCM stream and ODAS events, then serves aligned outputs through threaded server logic and socket-based frame emission.
- `audio_stream/magic/utils.py`: Shared utilities for cleanup, process termination, temp-file management, config patching, signal registration, and robust shutdown behavior across magic pipeline components.

- `audio_stream/old/chatak_audio_stream0111.py`: Early streaming implementation with live bandpass filtering class, update checks, and generated audio output for prototype web streaming experiments.
- `audio_stream/old/chatak_audio_stream0911.py`: Legacy streaming revision adding port management and WAV header handling around live bandpass filtering for browser-compatible audio transport.
- `audio_stream/old/chatak_audio_stream1011.py`: Iterative stream variant refining filter update flow and WAV generation while preserving Flask-based real-time audio endpoint behavior.
- `audio_stream/old/chatak_audio_stream1011works.py`: Known-working historical snapshot of the stream pipeline used as rollback reference during subsequent experiments on filtering and output formatting.
- `audio_stream/old/chatak_audio_stream1211.py`: Later old-version stream script extending prior filter-and-stream logic with additional operational tweaks for stability testing.
- `audio_stream/old/magicParser.py`: Older parser translating ODAS flat records into frame-indexed track and potential-source buckets for socket emission during pre-refactor magic pipeline stages.
- `audio_stream/old/odas_archive.py`: Historical ODAS archive parser and formatter that emits frame events, kept for compatibility with earlier consumers and behavioral regression checks.
- `audio_stream/old/pureMagicParser.py`: Simplified parser prototype focused on normalization of track and potential-source structures without full emission pipeline complexity.
- `audio_stream/old/sync_stream.py`: Previous synchronized stream server implementation coordinating queued audio with event emission, superseded by newer modular sync_stream architecture.
- `audio_stream/old/timmed_odas.py`: Time-based ODAS emitter utility that parses blocks and streams events with timing control for simulation, testing, and playback alignment experiments.

