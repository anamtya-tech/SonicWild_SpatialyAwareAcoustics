# ChatakGUI Config Database

Source: `ChatakGUI/config` from `SonicWild_RPi_Setup`.

- `bandpass.cfg`: Stores low and high frequency cutoffs used by audio filtering logic, currently configured to pass nearly full-band audio from 20 Hz to 20 kHz.
- `device_id.txt`: Holds the numeric device identifier used to tag recordings, telemetry, or uploaded artifacts with the specific deployed unit identity.
- `latlong.txt`: Stores deployment latitude and longitude metadata for the site, helping map or annotate recordings with geolocation context.
- `mic_location.txt`: Stores microphone placement coordinates, typically used to represent installation position separately from broader site coordinates.
- `pause.txt`: Stores a pause control value used by runtime scripts and GUI logic to represent paused state or pause duration behavior.
- `playback_start.txt`: Stores playback start marker or toggle value indicating when playback should begin in replay or analysis workflows.
- `playback_stop.txt`: Stores playback stop marker or toggle value indicating when playback should end in replay or analysis workflows.
- `project_name.txt`: Stores the active project label used for organizing session directories, naming outputs, and grouping deployment artifacts.
- `record.txt`: Stores recording enable state, used by automation scripts to decide whether capture should run or remain disabled.
- `sleep.txt`: Stores scheduled sleep time for device behavior or automation windows, formatted as a clock time.
- `wake.txt`: Stores scheduled wake time for device behavior or automation windows, formatted as a clock time.
- `xyz.txt`: Stores 3-axis coordinate values for position or orientation metadata, currently represented as comma-separated X, Y, Z values.
