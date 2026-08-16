# Troubleshooting — MT4 Trading

## Cheat sheet
Ports: 9443 HTTPS | 9090 HTTP (legacy) — both Tailscale-interface only
Logs: `tmux -S /home/tom/.tmux-monitor.sock attach -t monitor`
Containers: mt4-desktop, mt4-https-proxy

## Symptom -> Cause -> Fix
1. `docker ps` shows containers Up but PORTS column empty
   Docker started before Tailscale had assigned 100.75.220.84, so
   binding to that specific IP silently failed. -> confirm
   `/etc/systemd/system/docker.service.d/wait-for-tailscale.conf`
   exists (ExecStartPre loops until `tailscale ip -4` succeeds).
   Immediate fix: `docker compose down && docker compose up -d`.

2. `sudo reboot` powers the machine off but it never comes back
   iTCO_wdt hardware watchdog interferes with the ACPI reboot handoff
   on this chipset (`journalctl -b -1` shows "watchdog did not stop!").
   -> `echo "blacklist iTCO_wdt" | sudo tee /etc/modprobe.d/blacklist-watchdog.conf`
   then `sudo update-initramfs -u`.

3. `wine: '...' is a 64-bit installation, it cannot be used with a
   32-bit wineserver`
   Wine prefix is 64-bit; plain `wine` binary defaults to 32-bit.
   -> use `/usr/lib/wine/wine64` explicitly (find via
   `dpkg -L wine64 | grep bin` if path changes after a rebuild).

4. Container recreated/rebuilt and Wine is gone, MT4 won't launch
   Wine lives in image layers, not the bind-mounted /config — a fresh
   container from an old (pre-Dockerfile) image won't have it.
   -> confirm using `mt4-desktop-custom:latest` (built from the repo's
   Dockerfile), not the bare `linuxserver/webtop` image.

5. Container deleted accidentally (e.g. via `docker system prune`)
   Check `/home/tom/docker/mt4-server/config` first — if it's a bind
   mount (it is, by design), the Wine prefix + MT4 install + broker
   session survive on disk regardless. Just
   `docker compose up -d` to recreate the container; nothing lost.
   NEVER run `docker system prune -af --volumes` without excluding
   or manually confirming MT4 containers are not in the stop/remove
   list first.

6. HTTPS cert errors / can't reach :9443
   Tailscale cert issuance occasionally hits a known upstream bug
   (500 error on the ACME DNS-01 challenge, tailscale/tailscale#19942).
   -> `tailscale serve reset && tailscale down && tailscale up`
   (needs `sudo tailscale set --operator=tom` once, to run without
   sudo), then retry `sudo tailscale cert psth1.gila-karat.ts.net`.

7. KasmVNC window only fills part of the screen
   -> in-session settings (side tab, gear icon) -> Screen/Display ->
   set "Remote Resizing" (not fixed resolution / local scaling).
