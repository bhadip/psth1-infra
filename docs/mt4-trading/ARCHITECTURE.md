# Architecture — MT4 Trading

## Data flow
1. `mt4-desktop` container (custom image, built from
   `lscr.io/linuxserver/webtop:debian-openbox`) runs a full desktop via
   KasmVNC + Xvnc.
2. Wine + wine64 + tmux baked into the image at build time (Dockerfile).
3. MT4 (`terminal.exe`) auto-launches via an s6 service
   (`/etc/services.d/mt4/run`) — starts on container boot, s6 restarts
   it if it crashes.
4. `mt4-https-proxy` (Caddy) sits in front, terminates HTTPS using a
   Tailscale-issued cert, reverse-proxies to `mt4-desktop:3000`
   internally over plain HTTP (safe — never leaves the Docker network).
5. Both containers bind ONLY to the Tailscale interface
   (`100.75.220.84`), never the LAN IP — no exposure outside the tailnet.

## Services & boot chain
| What             | Mechanism                          | Auto-start |
|------------------|-------------------------------------|------------|
| docker.service   | systemd, waits for tailscaled IP    | yes        |
| mt4-desktop      | docker compose, restart:unless-stopped | yes     |
| MT4 terminal.exe | s6 service inside container         | yes        |
| mt4-https-proxy  | docker compose, restart:unless-stopped | yes     |
| tmux-monitor     | systemd, tails `docker logs -f`     | yes        |

## Ports (Tailscale interface only, 100.75.220.84)
9443/tcp HTTPS (KasmVNC/MT4) - 9090/tcp HTTP (legacy, still bound)

## Key design decisions
- Bind mounts (not named volumes) for `/config` — survived an
  accidental `docker system prune -af --volumes` intact. Named
  volumes would not have.
- Wine baked into the image (not installed post-hoc) — a container
  recreate/reboot no longer needs a ~15-min manual reinstall.
- Two containers (desktop vs. HTTPS proxy), not one — `webtop` base
  image only speaks plain HTTP; bolting TLS into it directly would be
  fragile and get overwritten on rebuild.
- Tailscale-only binding, not Cloudflare Tunnel — MT4 login/session
  must have zero public exposure path, not "public but authenticated."
  See DECISIONS.md.
- `wine64` must be invoked explicitly — the `.wine` prefix is a 64-bit
  install; plain `wine` defaults to the 32-bit wineserver and fails.

## Known limitations
- iTCO_wdt hardware watchdog on this chipset previously broke clean
  reboot/auto-poweron — fixed by blacklisting the module (see
  TROUBLESHOOTING.md). Confirm this stays blacklisted after any
  kernel/initramfs rebuild.
- Broker login is entered manually via the KasmVNC GUI, never scripted
  or stored in any file/command — deliberate, to keep credentials out
  of shell history and backups.

## Planned: multiple concurrent accounts
Capacity confirmed sufficient (i5-3320M ~90% idle, 2.7GB RAM free) for
2-3 concurrent MT4 accounts, same broker. Plan: copy the install folder
inside the same Wine prefix (`MetaTrader 4 - Account2`, `...Account3`),
one s6 service per account, one tmux window per account's log. Same
Tailscale URL — appears as multiple windows on one desktop, not
separate connections. Not yet implemented.
