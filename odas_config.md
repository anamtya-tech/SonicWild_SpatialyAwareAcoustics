# ODAS Config File Summary

Source folder: `odas/build/bin/config` (from `SonicWild_RPi_Setup`).

- `1City_DTNSC_respeaker_usb_4_mic_array.cfg`: City-profile ODAS configuration variant for ReSpeaker USB 4-mic array, tuned for DTNSC processing and localization/tracking flow in the SonicWild pipeline.
- `1_band_pass_10_300.cfg`: Band-pass ODAS profile focused on very low frequencies (10-300 Hz) for analysis of low-end acoustic content with standard six-channel raw microphone input.
- `2City_DTNSC_respeaker_usb_4_mic_array.cfg`: Secondary city-profile DTNSC configuration referencing recorded raw session paths under `/mnt/CHATAK_VM/Live_Audio` for repeatable playback-based ODAS processing.
- `2_band_pass_300_700.cfg`: Band-pass ODAS profile for low-mid frequencies (300-700 Hz), useful for isolating lower vocal or environmental bands during localization experiments.
- `3_band_pass_700_1500.cfg`: Band-pass ODAS profile covering mid frequencies (700-1500 Hz), typically used for directional clarity and source discrimination in mixed outdoor soundscapes.
- `4_band_pass_1500_4000.cfg`: Band-pass ODAS profile for upper-mid frequencies (1500-4000 Hz), targeting sharper transients and detail-rich content in directional tracking workflows.
- `5_band_pass_4000_8000.cfg`: Band-pass ODAS profile for high frequencies (4000-8000 Hz), often used to study high-energy components and improve source separation behavior.
- `6_band_pass_8000_15000.cfg`: Band-pass ODAS profile for very high frequencies (8000-15000 Hz), supporting experiments focused on fine high-frequency directionality response.
- `archive.cfg`: Archive-playback ODAS configuration that reads a specific recorded RAW session path and runs standard mapping, SSL, and tracking stages for offline processing.
- `City_DTNSC_respeaker_usb_4_mic_array.cfg`: Main city DTNSC ReSpeaker profile tied to a live-audio RAW session file path for deterministic ODAS replay and tuning.
- `City_DTNSC_respeaker_usb_4_mic_array_20251223.cfg`: Timestamped backup/snapshot of city DTNSC ReSpeaker configuration preserving a known-good parameter set from the dated tuning cycle.
- `DTNSC_respeaker_usb_4_mic_array.cfg`: General DTNSC ReSpeaker baseline configuration with optional raw file path comments, used as a core template for live or file-driven ODAS runs.
- `Forest_DTNSC_respeaker_usb_4_mic_array.cfg`: Forest-environment DTNSC profile for ReSpeaker input, intended for outdoor woodland acoustic conditions and scenario-specific localization calibration.
- `org_respeaker_usb_4_mic_array.cfg`: Original/reference ODAS ReSpeaker configuration retained as baseline before environment-specific and band-pass variants were introduced.
- `rec_City_.cfg`: Recorded-city playback configuration targeting specific stored RAW session data for reproducible city-noise ODAS testing and comparative evaluation.
- `Socket.cfg`: Socket-input ODAS configuration variant using network parameters (port-based endpoint) for streamed data ingestion instead of direct device or file path input.
