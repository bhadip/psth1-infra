# Architecture — MT4 Trading

## Data flow
1. `mt4-desktop` container (custom image, built from `lscr.io/linuxserver/webtop:debian-openbox`) runs a full desktop via KasmVNC + Xvnc.
2. Wine + wine64 + tmux baked into the image at build time (Dockerfile).
3. **Multiple MT4 Instances**: MT4 (`terminal.exe`) auto-launches via separate s6 services (`/etc/services.d/mt4/run` and `/etc/services.d/mt4-ct/run`). Each points to a distinct installation folder (`MetaTrader 4` and `MetaTrader 4 - CT`) within the same Wine prefix, allowing 2+ concurrent accounts on the same desktop.
4. `mt4-https-proxy` (Caddy) sits in front, terminates HTTPS using a Tailscale-issued cert, reverse-proxies to `mt4-desktop:3000` internally over plain HTTP.
5. Both containers bind ONLY to the Tailscale interface (`100.75.220.84`), never the LAN IP.

## Services & boot chain
| What | Mechanism | Auto-start |
| --- | --- | --- |
| docker.service | systemd, waits for tailscaled IP | yes |
| mt4-desktop | docker compose, restart:unless-stopped | yes |
| MT4 Instance 1 | s6 service (`/etc/services.d/mt4/run`) | yes |
| MT4 Instance 2 (CT) | s6 service (`/etc/services.d/mt4-ct/run`) | yes |
| mt4-https-proxy | docker compose, restart:unless-stopped | yes |
| tmux-monitor | systemd, tails `docker logs -f` | yes |

## Ports (Tailscale interface only, 100.75.220.84)
- `9443/tcp`: HTTPS (KasmVNC/MT4)
- `9090/tcp`: HTTP (legacy, still bound)

## Key design decisions
- **Bind mounts** (not named volumes) for `/config` — survived accidental `docker system prune -af --volumes` intact.
- **Wine baked into the image** — container recreate/reboot doesn't need a ~15-min manual reinstall.
- **Two containers** (desktop vs. HTTPS proxy) — `webtop` base image only speaks plain HTTP; bolting TLS into it directly is fragile.
- **Tailscale-only binding** — MT4 login/session has zero public exposure path.
- **`wine64` invoked explicitly** — the `.wine` prefix is a 64-bit install; plain `wine` defaults to 32-bit and fails.
- **Concurrent Accounts** — Achieved by duplicating the MT4 folder (e.g., `MetaTrader 4 - CT`) and adding a second `s6` service. They share the same KasmVNC desktop but run as independent `terminal.exe` processes.

## Known limitations
- iTCO_wdt hardware watchdog previously broke clean reboot — fixed by blacklisting the module. Confirm this stays blacklisted after kernel/initramfs rebuilds.
- Broker logins are entered manually via the KasmVNC GUI, never scripted, to keep credentials out of shell history and backups.
- **KasmVNC Clipboard Bug**: The browser-based VNC client can crash with a "Kasm encountered an error" when switching windows or pasting. **Fix**: Open the Kasm sidebar settings and turn OFF "Clipboard Seamless", "Clipboard Up", and "Clipboard Down".
