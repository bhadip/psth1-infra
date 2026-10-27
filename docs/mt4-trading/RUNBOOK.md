# Runbook — MT4 Trading

## Access
`https://psth1.gila-karat.ts.net:9443` (Tailscale must be connected)
*(Note: Use only ONE browser tab. Both MT4 instances run inside the same virtual desktop).*

## KasmVNC Clipboard Crash Fix
If you see "Kasm encountered an error" when switching windows:
1. Open the Kasm sidebar menu (left edge of screen).
2. Go to **Settings**.
3. Turn **OFF**: `Clipboard Seamless`, `Clipboard Up`, and `Clipboard Down`.
4. Hard refresh the browser (`Ctrl+Shift+R`).

## Check status
```bash
docker ps                                        # both containers Up?
docker exec mt4-desktop ps aux | grep terminal.exe # Should show 2+ terminal.exe processes
tmux -S /home/tom/.tmux-monitor.sock attach -t monitor   # live MT4 log
```

## Restart MT4 only (not the whole container)
```bash
# Instance 1
docker exec -u root mt4-desktop s6-svc -d /etc/services.d/mt4   # stop
docker exec -u root mt4-desktop s6-svc -u /etc/services.d/mt4   # start

# Instance 2 (CT)
docker exec -u root mt4-desktop s6-svc -d /etc/services.d/mt4-ct   # stop
docker exec -u root mt4-desktop s6-svc -u /etc/services.d/mt4-ct   # start
```

## Add a New MT4 Instance (e.g., Account 3)
1. Duplicate the folder inside the container:
   ```bash
   docker exec -u root mt4-desktop cp -r "/config/.wine/drive_c/Program Files (x86)/MetaTrader 4" "/config/.wine/drive_c/Program Files (x86)/MetaTrader 4 - New"
   docker exec -u root mt4-desktop chown -R abc:abc "/config/.wine/drive_c/Program Files (x86)/MetaTrader 4 - New"
   ```
2. Create the s6 service:
   ```bash
   docker exec -u root mt4-desktop mkdir -p /etc/services.d/mt4-new
   docker exec -u root mt4-desktop sh -c "cat > /etc/services.d/mt4-new/run << 'INNEREOF'
#!/usr/bin/with-contenv bash
sleep 20
exec su abc -c \"/usr/lib/wine/wine64 '/config/.wine/drive_c/Program Files (x86)/MetaTrader 4 - New/terminal.exe'\"
INNEREOF"
   docker exec -u root mt4-desktop chmod +x /etc/services.d/mt4-new/run
   ```
3. Create desktop shortcut:
   ```bash
   docker exec -u root mt4-desktop sh -c "mkdir -p /home/abc/Desktop && cat > /home/abc/Desktop/MT4-New.desktop << 'INNEREOF'
[Desktop Entry]
Name=MT4 - New Account
Exec=/usr/lib/wine/wine64 '/config/.wine/drive_c/Program Files (x86)/MetaTrader 4 - New/terminal.exe'
Type=Application
Icon=application-vnd.wine-extension-exe
INNEREOF"
   docker exec -u root mt4-desktop chown abc:abc /home/abc/Desktop/MT4-New.desktop
   ```
4. Restart container to apply: `docker restart mt4-desktop`

## Rebuild the image (after Dockerfile changes)
```bash
cd /home/tom/docker/mt4-server
docker compose build
docker compose up -d
```

## Reboot behaviour
Auto-recovers fully unattended: Docker waits for Tailscale IP; mt4-desktop has `restart:unless-stopped`; both MT4 instances auto-launch via s6.

## Wake psth1 remotely (if powered off)
See `../networking/RUNBOOK.md` — Wake-on-LAN section.
