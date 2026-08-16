# Decisions — why things were built this way

## Tailscale AND Cloudflare, not one or the other
Cloudflare Access/WARP could technically replace Tailscale, but the
security model differs: Tailscale = no public exposure path exists at
all (must already be an authorized tailnet device). Cloudflare Access
= publicly reachable, gated by an auth check. For a live trading
terminal, "no door exists" was judged worth more than "door exists,
but locked." Also avoids a single-vendor point of failure, and keeps
the Raspberry Pi relay plan viable (depends on Tailscale mesh reach).

## Separate concerns, always, even when one tool "could" do it all
Recurring pattern across this project: Portainer+NPM over Cosmos
(one tool trying to do container mgmt + proxy + firewall + auth in one
process = fragile); Caddy + cloudflared as distinct containers, not
one; mt4-desktop vs mt4-https-proxy split. Lesson learned directly
from the Cosmos incident (see LIMITATIONS.md / the MT4 recovery
story) — one thing doing too much is where instability came from.

## Bind mounts over named Docker volumes, always, for anything that
must survive a container/host lifecycle event
Directly saved the MT4 install during an accidental
`docker system prune -af --volumes` — the container was deleted, the
bind-mounted config folder was not.

## New "home services" workloads get their own hardware, not psth1
psth1 is protected for MT4's sake. Anything not required for trading
(downloader, PXE server, heavier monitoring) is deliberately scoped to
a separate, cheap, low-power box rather than added here — same reason
Cosmos's resource competition was a problem in the first place.

## Local-first + client-side-encrypted backup, not direct-to-OneDrive
Bambang's explicit, stated priority: privacy, resilience, and control
— specifically not trusting that files sent directly to OneDrive can't
be opened/copied by others. Files live on owned hardware; OneDrive
only ever receives already-encrypted blobs (rclone crypt / restic),
and is treated as a backup copy, never the source of truth.

## restic (not plain `rclone sync`) for the OneDrive backup
Plain sync only mirrors — no history, accidental deletions propagate
immediately. restic adds true versioned/prunable snapshots + dedup +
integrity checking on top of the same encrypted-transport goal.
