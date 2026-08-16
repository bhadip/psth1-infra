# Architecture — Networking (Tailscale, AdGuard, WoL, Router)

## Topology (double NAT — important, non-obvious)
    Fiber -> ZTE GPON ONR (192.168.1.254, TRUE gateway, no admin access)
               |-- (previously) switch -> psth1
               '-- AX3000 (own separate router+NAT, 192.168.50.x)
psth1 was moved from the ONR's own segment onto AX3000's LAN
(192.168.50.2, DHCP-reserved) specifically so Wake-on-LAN and AdGuard
DNS enforcement work for AX3000-connected devices without needing ONR
access (which Bambang does not have).

## Tailscale
- Zero-public-exposure private mesh. Devices: psth1, MacBook, Windows
  desktop, Android phone.
- Used for: MT4/KasmVNC (:9443), AdGuard dashboard (:3000), SSH to
  psth1, and (optionally) subnet-routing into 192.168.50.x for
  reaching the AX3000 router's SSH while away from home.
- `sudo tailscale cert <hostname>` issues real browser-trusted certs
  for internal HTTPS services — see mt4-trading/TROUBLESHOOTING.md for
  a known upstream bug affecting this command.

## AdGuard Home (DNS-level filtering + logging)
- Container `adguard-home`: DNS (port 53) bound to `192.168.50.2`
  (LAN-wide, needed by all devices).
- Container `adguard-https-proxy` (Caddy, reuses the Tailscale cert):
  dashboard (port 3000) bound ONLY to `100.75.220.84` (Tailscale) —
  deliberately not reachable from plain LAN.
- AX3000 DHCP -> DNS Server 1 = 192.168.50.2 (AdGuard), DNS Server 2 =
  94.140.14.14 (public AdGuard, fallback if psth1 is down). Stock
  firmware only supports 2 DHCP DNS entries — no 3rd fallback possible.
- Custom blocklist maintained in a public GitHub repo
  (raw.githubusercontent.com URL added under Filters -> DNS
  blocklists) — AdGuard auto-refetches on its own schedule; editable
  from any device including mobile via GitHub's app.

## Wake-on-LAN
- Requires: BIOS-level "Wake on LAN" enabled (was OFF by default on
  this hardware — this was the actual root cause of early failures,
  not a networking issue) + OS-level `ethtool -s <iface> wol g`.
- Magic packets are Layer-2 broadcast only — cannot cross routers/NAT.
  This is why psth1 had to physically move onto AX3000's own LAN
  segment; WoL from AX3000-connected devices did not work while psth1
  sat on the ONR's separate segment.
- True remote wake (from outside the house) additionally requires
  something ALWAYS-ON and independent of psth1's power state, since
  psth1 itself can't relay a wake request for itself while off.
  Tested: Tailscale subnet-routing through psth1 to reach the router's
  SSH does NOT solve this (psth1 being off removes the only path in).
  Planned fix: a Raspberry Pi running its own independent Tailscale
  connection, sitting on the 192.168.50.x segment — location currently
  unknown (storeroom), not yet set up.

## Router (Asus RT-AX3000P) constraints
- Stock, ISP-locked "P" variant firmware.
- Asuswrt-Merlin NOT available for this model (different hardware
  from the retail RT-AX3000 V1, confirmed via Merlin's official
  supported-device list — do not attempt to flash it).
- JFFS custom scripts NOT available (`jffs2_scripts` nvram variable
  doesn't exist on this firmware) — router-level DNS-hijack /
  force-all-devices-through-AdGuard is not achievable here.
- SSH enabled, LAN-only (Administration -> System). Router's own
  VPN server (if present) can't be reached from outside anyway — port
  forwarding for it would need to live on the ONR, which isn't
  accessible.

## Known limitations
- Tailscale on the Android phone showed degraded browsing (some URLs
  failing/slow) — likely a DERP relay fallback over mobile carrier
  NAT rather than a direct P2P path. Not resolved; mitigated by NOT
  depending on Tailscale for at-home DNS (LAN-direct instead) and not
  routing Syncthing through it.
- No away-from-home AdGuard filtering for the household's non-psth1
  devices (only achievable via the abandoned JFFS/router-hijack path).
