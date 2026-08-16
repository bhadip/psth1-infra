# psth1 Infrastructure

######
One small non-security note, just for your awareness, not a blocker:
This file references an absolute host path (/home/tom/docker/mt4-server/certs) rather than a relative one — meaning if you ever restore this repo onto a different machine (or a fresh psth1 after a full reinstall), that exact path would need to exist for the container to start. Not a secret leak, just a portability wrinkle worth knowing about — not worth fixing right now, but good to remember if this repo is ever used to rebuild psth1 from scratch someday.
######

Home server (Dell Latitude E5432, Debian 13, 4GB RAM) running live forex
trading, a public website, DNS-level ad/tracker blocking, and remote
access — alongside the DVR (see /dvr/docs). Host: `psth1`,
Tailscale `100.75.220.84`, LAN `192.168.50.2`, tailnet `gila-karat.ts.net`.

## Subsystems
- `mt4-trading/`  — MT4 live trading terminal (Wine + Docker + KasmVNC)
- `prasanti-web/` — prasanti.com public site (Cloudflare Tunnel + Caddy)
- `networking/`   — Tailscale, AdGuard Home, Wake-on-LAN, router config

## Cross-cutting docs
- `DECISIONS.md`    — why things were built the way they were
- `LIMITATIONS.md`  — known gaps, not bugs, don't re-chase these

## Priority
MT4 (live trading) always takes resource/reliability priority over
everything else on this host. New services are deliberately kept off
psth1 (separate hardware) once they'd compete with it.

## Quick links
- MT4/KasmVNC:  https://psth1.gila-karat.ts.net:9443  (Tailscale only)
- AdGuard:      https://psth1.gila-karat.ts.net:3000  (Tailscale only)
- prasanti.com: https://www.prasanti.com              (public)
