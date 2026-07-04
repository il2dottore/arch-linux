# Fix memory leak after Suspend/Resume
```bash
sudo mkdir -p /etc/wireplumber/wireplumber.conf.d
sudo nano /etc/wireplumber/wireplumber.conf.d/20-intel-audio.conf
```
```bash
wireplumber.profiles = {
  main = {
    monitor.alsa = required
    monitor.alsa-midi = disabled
  }
}

monitor.alsa.rules = [
  {
    matches = [
      {
        device.name = "alsa_card.pci-0000_00_1f.3"
      }
    ]
    actions = {
      update-props = {
        api.alsa.use-acp = false
        api.alsa.use-ucm = false
      }
    }
  }
]
```
# Fix losing volume hardware settings after reboot.
```bash
sudo nano /etc/systemd/user/restore-volume-settings.service
```
```bash
[Unit]
Description=Set Intel PCH mixer levels after WirePlumber
After=wireplumber.service
Requires=wireplumber.service
PartOf=wireplumber.service

[Service]
Type=oneshot
ExecStart=/usr/bin/amixer -c PCH sset Master 100%% unmute
ExecStart=/usr/bin/amixer -c PCH sset Headphone 100%% unmute
ExecStart=/usr/bin/amixer -c PCH sset Speaker 100%% unmute
ExecStart=/usr/bin/amixer -c PCH sset PCM 100%%
ExecStart=/usr/bin/amixer -c PCH sset IEC958,0 unmute
ExecStart=/usr/bin/amixer -c PCH sset IEC958,1 unmute
ExecStart=/usr/bin/amixer -c PCH sset IEC958,2 unmute
ExecStart=/usr/bin/amixer -c PCH sset IEC958,3 unmute
ExecStart=/usr/bin/amixer -c PCH sset "Auto-Mute Mode" Enabled
ExecStart=/usr/bin/amixer -c PCH sset "Headset Mic Boost" 100%%
ExecStart=/usr/bin/amixer -c PCH sset "Internal Mic Boost" 100%%
RemainAfterExit=yes

[Install]
WantedBy=default.target
```
```bash
systemctl --user enable --now restore-volume-settings.service
```
