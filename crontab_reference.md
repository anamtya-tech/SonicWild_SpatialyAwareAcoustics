# Crontab Reference

Source: `cronjobs.txt` from `SonicWild_RPi_Setup`.

## User crontab (`crontab -e`)

- `@reboot /usr/bin/python3 /home/chatak/ChatakGUI/record.py`: Starts the main recording service automatically on boot so audio capture begins without manual intervention.
- `@reboot cd /mnt/CHATAK_VM && python3 cors_http_server.py`: Launches the CORS-enabled HTTP server from mounted storage to expose files and endpoints required by frontend access.
- `@reboot sleep 10 && /usr/bin/python3 /home/chatak/ChatakGUI/record.py`: Delayed fallback startup for recorder, useful when storage or audio devices need extra boot time before recording begins.

## Root crontab (`sudo crontab -e`)

- `@reboot /usr/bin/python3 /home/chatak/ChatakGUI/`: Placeholder/incomplete command in current notes; likely intended to run a specific script, not a directory path.
- `@reboot /usr/bin/python3 /home/chatak/ChatakGUI/asound_check.py`: Auto-runs ALSA/ReSpeaker detection and updates sound configuration to keep audio routing valid after reboot.
- `@reboot /usr/bin/python3 /home/chatak/ChatakGUI/usb_mount.py`: Runs USB or remote mount setup at startup to ensure storage paths are ready before recording and server tasks execute.

## Recommended Notes

- Avoid duplicate recorder startup lines unless fallback behavior is intentional.
- Replace directory-only command with an explicit script path.
- Keep absolute interpreter and script paths to reduce boot-time environment issues.
