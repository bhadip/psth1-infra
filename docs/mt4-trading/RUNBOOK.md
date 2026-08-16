# Runbook — MT4 Trading

## Access
    https://psth1.gila-karat.ts.net:9443    # Tailscale must be connected

## Check status
    docker ps                                        # both containers Up?
    docker exec mt4-desktop ps aux | grep terminal.exe
    tmux -S /home/tom/.tmux-monitor.sock attach -t monitor   # live MT4 log

## Restart MT4 only (not the whole container)
    docker exec -u root mt4-desktop s6-svc -d /run/service/mt4   # stop
    docker exec -u root mt4-desktop s6-svc -u /run/service/mt4   # start

## Rebuild the image (after Dockerfile changes)
    cd /home/tom/docker/mt4-server
    docker compose build      # ~15 min (Wine install), add BuildKit
                               # cache mounts to speed up repeat builds
    docker compose up -d

## Reboot behaviour
Auto-recovers fully unattended: Docker waits for Tailscale IP before
starting containers; mt4-desktop has restart:unless-stopped; MT4
auto-launches via s6; tmux-monitor.service recreates the log session.
Verified via full reboot test — no manual steps required.

## Wake psth1 remotely (if powered off)
See ../networking/RUNBOOK.md — Wake-on-LAN section.

## Editing rules
- Any change to the Dockerfile or Caddyfile requires
  `docker compose down && docker compose up -d` to take effect —
  `restart` alone does not re-read compose file changes.
