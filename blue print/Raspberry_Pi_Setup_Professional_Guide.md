# Raspberry Pi Mic Array Setup Guide

## Purpose
This guide documents the Raspberry Pi setup flow for the SonicWild edge audio stack, including OS provisioning, remote access, file deployment, audio stack configuration, auto-start services, storage mount, and validation.

## Scope
- Target device: Raspberry Pi (hostname: `chatak`)
- Main workload: passive multichannel audio capture and edge services
- Related components: `ChatakGUI`, `z_odas`, `cors_http_server.py`, mounted flash storage

## Prerequisites
- Raspberry Pi with network connectivity
- MicroSD card with Raspberry Pi OS
- Laptop/desktop with SSH and SCP available
- Setup files prepared locally (example base folder: `D:/RPi_Setup/`)

## 1) Flash Raspberry Pi OS
During image configuration, use:
- Hostname: `chatak`
- Username/password: `chatak` / `chatak`
- Wi-Fi SSID: `sonic wild`
- Enable SSH: `Yes`

Reference video:
- https://www.youtube.com/watch?v=RVosaVGXQSk

## 2) Enable VNC on Raspberry Pi
Enable VNC in Raspberry Pi configuration for GUI remote access.

Reference video:
- https://www.youtube.com/watch?v=h3YHOhyjmJk

## 3) Install RealVNC Viewer and Connect
Install RealVNC Viewer on your laptop and connect to the Pi using the configured hostname/IP.

Reference video:
- https://www.youtube.com/watch?v=4pJMfvaNgaw

## 4) Copy Setup Files
Copy project files from local setup folder to Pi home:

```bash
# Example target
/home/chatak/
```

Copy system config files:

```bash
# ALSA config
sudo cp /home/chatak/asound.conf /etc/asound.conf

# Wi-Fi supplicant config
sudo cp /home/chatak/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant.conf
```

## 5) SCP Command Examples
Use `<ip>` with your Pi IP address.

```bash
# Download file from Pi
scp chatak@<ip>:/etc/wpa_supplicant/wpa_supplicant.conf D:/RPi_Setup/

# Upload file to Pi
scp D:/RPi_Setup/wpa_supplicant.conf chatak@<ip>:/etc/wpa_supplicant/

# Download folder from Pi
scp -r chatak@<ip>:/home/chatak/ChatakGUI D:/RPi_Setup/

# Upload folder to Pi
scp -r D:/RPi_Setup/ChatakGUI chatak@<ip>:/home/chatak/
```

## 6) Install ODAS (`z_odas`)
Install and build ODAS-based localization stack.

Reference article:
- https://medium.com/@bharathsudharsan023/odas-open-embedded-audition-system-sound-source-localization-tracking-separa

## 7) Install PulseAudio and Configure ALSA

```bash
sudo apt update
sudo apt install -y pulseaudio
sudo nano /etc/asound.conf
```

## 8) Configure Auto-Start (crontab)

```bash
crontab -e
```

Add:

```cron
@reboot /usr/bin/python3 /home/chatak/ChatakGUI/record.py
@reboot cd /mnt/CHATAK_VM && python3 cors_http_server.py
```

## 9) Mount Flash Drive via SSHFS

```bash
sudo apt update
sudo apt install -y sshfs
mkdir -p ~/CHATAK_VM

# Mount remote storage
sshfs chatak@chatak:/mnt/CHATAK_VM ~/CHATAK_VM
ls ~/CHATAK_VM
```

Persist mount in `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add:

```fstab
chatak@chatak:/mnt/CHATAK_VM /home/pi/CHATAK_VM fuse.sshfs defaults,_netdev,users,idmap=user,IdentityFile=/home/pi/.ssh/id_rsa 0 0
```

Apply:

```bash
sudo mount -a
```

## 10) Deploy Server Script
Copy `cors_http_server.py` to mounted drive location:

```bash
/mnt/CHATAK_VM/
```

## 11) Validate Audio Capture

```bash
arecord -D plug:multi_capture -c 6 -r 16000 -f S16_LE -d 5 /mnt/CHATAK_VM/TEST.wav
```

Validation checklist:
- `TEST.wav` is created under `/mnt/CHATAK_VM/`
- expected background services/processes are running

## Operational Outcome
When completed, the Pi is ready for:
- passive multichannel audio capture
- remote access and management
- mounted storage sync
- server-side data/service deployment

## Troubleshooting Quick Checks
- Verify SSH connectivity: `ssh chatak@<ip>`
- Confirm mount status: `mount | grep CHATAK_VM`
- Confirm audio device path in ALSA config: `/etc/asound.conf`
- Re-check crontab entries: `crontab -l`
