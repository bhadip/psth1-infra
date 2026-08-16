# Limitations — known gaps, don't re-chase these

## Networking
- No away-from-home AdGuard filtering for non-psth1 devices. Router
  firmware can't force it (no JFFS scripting); Tailscale-push-DNS
  approach shelved due to mobile relay slowness.
- No true remote Wake-on-LAN yet (needs the still-unlocated Raspberry
  Pi as an independent always-on relay). Local-segment WoL works.
- No admin access to the actual internet gateway (ZTE GPON ONR) — the
  AX3000 sits behind it in a double-NAT. AX3000's own security log
  can't see traffic to/from psth1 at all (architecturally invisible to
  it); the ONR would be the correct place to look, but isn't
  accessible.
- Asuswrt-Merlin firmware is NOT available for the RT-AX3000P (an
  earlier session assumption that Merlin worked on the *RT-AC2600* was
  also wrong and was corrected — neither router should be flashed).
- Full network traffic visibility (bandwidth per device, "what media
  was accessed") is not achievable without inline routing or a
  mirror-port-capable switch — deliberately not pursued (would need
  TLS interception, judged not worth the invasiveness/fragility for a
  home network). Domain-level visibility via AdGuard is the ceiling.

## Email / DNS
- DKIM not configured (needs Google Workspace Admin console step).
- DMARC is in monitoring mode only (`p=none`), not yet enforcing.

## MT4
- Only one account currently running live; 2nd/3rd account plan is
  designed and capacity-confirmed but not yet implemented.

## Backup / storage
- Syncthing + restic/rclone-crypt OneDrive backup is designed
  (see mt4-trading is unrelated; this belongs to a not-yet-created
  `backup/` doc set once implemented) but not yet built.

## Hardware
- D-Link DNS-323 confirmed not viable for anything modern (2007-era,
  ~500MHz MIPS, 32-64MB RAM) — retire/recycle, don't try to repurpose.
- Local LLM (Ollama) for coding-agent work is not viable on psth1 at
  all (no GPU). Would need separate, materially pricier hardware if
  ever pursued — not bundled into any home-server hardware plan.
