# Orange Pi Zero 512MB LTS — H3 Allwinner Setup Guide

## Image

- Board: Orange Pi Zero 512MB LTS (H3 Allwinner)
- Mirror: https://xogium.performanceservers.nl/archive/orangepizero/archive/
- Image: `Armbian_25.5.1_Orangepizero_noble_current_6.12.23_minimal.img.xz`

```bash
xz --decompress Armbian_25.5.1_Orangepizero_noble_current_6.12.23_minimal.img.xz
```

Flash to a 16GB SD card using Balena Etcher:
https://github.com/balena-io/etcher/releases/download/v2.1.7/balenaEtcher-linux-x64-2.1.7.zip

---

## First Boot

SSH is enabled by default.

```
user: root
pass: 1234
```

First boot forces you to change password and set up a user and locales.

```bash
sudo apt-get update
sudo apt-get upgrade
# Accept the SSH update prompt
reboot
```

Log back in as your user (not root):

```bash
sudo apt-get update
sudo apt-get full-upgrade
```

---

## Bluetooth

Insert USB Bluetooth dongle, then:

```bash
sudo apt install bluetooth bluez bluez-tools
```

Verify:

```bash
bluetoothctl
```

Install PipeWire audio stack:

```bash
sudo apt-get install pipewire pipewire-audio-client-libraries pipewire-pulse \
  pipewire-jack pipewire-alsa libspa-0.2-bluetooth wireplumber

sudo apt-get install libspa-0.2-bluetooth pipewire-audio

sudo apt install pulseaudio-utils
```

Enable user lingering so services run without a graphical session:

```bash
sudo loginctl enable-linger <your-user>
```

Start and enable PipeWire services as your user:

```bash
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```

### WirePlumber — disable seat monitoring (headless fix)

```bash
sudo nano /usr/share/wireplumber/wireplumber.conf
```

Add:

```
monitor.bluez.seat-monitoring = disabled

wireplumber.profiles = {
  main = {
    monitor.bluez.seat-monitoring = disabled
  }
}
```

### ALSA default to PipeWire

```bash
sudo nano ~/.asoundrc
```

Add:

```
pcm.!default {
    type pipewire
}

ctl.!default {
    type pipewire
}
```

---

## Wi-Fi

```bash
sudo apt install network-manager
sudo ip link set wlan0 up
sudo nmcli dev wifi connect "YourSSID" password "YourPassword"
sudo nmcli connection show
sudo nmcli connection modify "YourSSID" connection.autoconnect yes
```

---

## OSMC — Open Multi Speaker Connect

```bash
sudo apt install libportaudio2
sudo apt install python3-pip
sudo apt install python3-numpy python3-scipy
sudo apt install gcc libffi-dev python3-dev
pip install --break-system-packages "sounddevice>=0.4.6"
sudo apt install git
git clone https://github.com/harmen91/open-multi-speaker-connect.git
cd open-multi-speaker-connect
chmod +x tui.sh
```

---

## UxPlay — AirPlay Server

```bash
sudo apt install uxplay avahi-daemon \
  gstreamer1.0-tools gstreamer1.0-plugins-base \
  gstreamer1.0-plugins-good gstreamer1.0-pipewire
```

Enable Avahi for network discovery:

```bash
sudo systemctl enable --now avahi-daemon
```

Create the systemd service:

```bash
sudo nano /etc/systemd/system/uxplay.service
```

```ini
[Unit]
Description=UXPlay AirPlay Server
After=network.target sound.target

[Service]
User=orange
Environment=XDG_RUNTIME_DIR=/run/user/1000
Environment=DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
ExecStart=/usr/bin/uxplay
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Create UxPlay config:

```bash
nano ~/.uxplayrc
```

```
as pipewiresink
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now uxplay
sudo systemctl enable avahi-daemon uxplay
```

> **Note:** Use network (AirPlay) as the audio source rather than Bluetooth input to avoid USB bottleneck on single-controller boards like the H3.
