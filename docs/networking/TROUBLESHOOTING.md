# Troubleshooting — Networking

## Cheat sheet
psth1 LAN: 192.168.50.2 (reserved) | AX3000: 192.168.50.1
AdGuard: :53 (LAN) / :3000 (Tailscale only) | MT4: :9443 (Tailscale only)
Router SSH: LAN-only, prasanti@192.168.50.1 (or via Tailscale subnet route)

## Symptom -> Cause -> Fix
1. WoL packet sent, psth1 doesn't wake, even from same segment
   Check BOTH layers separately — OS setting alone is not enough:
   `sudo ethtool <iface> | grep Wake-on` must show `g`. If it does but
   still fails -> BIOS. Reboot, F2 at POST, Power Management -> enable
   "Wake on LAN" explicitly (was OFF by default on this hardware).

2. WoL works on same segment, not from elsewhere
   Expected — magic packets don't cross routers (Layer-2 broadcast
   only). Not fixable via router config alone. Needs an always-on
   relay device on the same segment as psth1 (planned: Raspberry Pi).

3. `ether-wake -b -i br0 ...` — command not found / doesn't work
   `-i` takes an INTERFACE name, not a MAC address. Interface is
   `br0` on this router (verify: `ip link show | grep br`), not the
   target MAC.

4. Can't SSH to AX3000 from outside the house at all
   SSH is LAN-only by design (security choice). Use Tailscale subnet
   routing through psth1 instead (see RUNBOOK.md) — only works while
   psth1 is on.

5. AdGuard dashboard unreachable at the LAN IP
   By design — dashboard is Tailscale-interface-only
   (100.75.220.84:3000), not LAN-reachable, to prevent any device on
   the WiFi from reaching the admin UI. Use
   https://psth1.gila-karat.ts.net:3000 over Tailscale instead.

6. AdGuard "Add blocklist" fails: "data is HTML, not plain text"
   URL pointed at a webpage, not a raw rules file.
   -> use a raw.githubusercontent.com URL, or Filters -> Custom
   filtering rules for one-off manual entries.

7. AdGuard "Add blocklist" fails: "path does not match safe patterns"
   AdGuard rejects arbitrary local filesystem paths, even ones inside
   its own mounted volume — not configurable. -> host the list on
   GitHub instead (see RUNBOOK.md).

8. Device ignores AdGuard even though DHCP DNS is set correctly
   Device has DNS-over-HTTPS/Private DNS active, bypassing DHCP-handed
   DNS entirely. -> Android: Settings -> Network & Internet ->
   Private DNS -> Off. No router-side fix exists on this firmware
   (JFFS scripting unavailable, confirmed).

9. Tailscale-connected phone: browsing slow / some URLs fail
   Likely DERP relay fallback (no direct P2P over mobile carrier NAT).
   `tailscale status` shows relay region instead of `direct`.
   Not resolved — workaround is to not depend on Tailscale for
   at-home DNS (works LAN-direct without it) and keep Tailscale off
   unless specifically needed (MT4 access, router SSH via subnet
   route).

10. Asuswrt-Merlin firmware page shows RT-AX3000P as unsupported
    Correct, not a mistake to work around. RT-AX3000P is a different
    board from the retail RT-AX3000 V1 despite the similar name. Do
    not attempt to flash Merlin (or any non-stock firmware) on this
    router — it's the live household gateway, high blast radius if a
    flash fails, and the ISP "P" variant may have a locked bootloader
    regardless.

11. DHCP reservation set but device gets a different IP than expected
    Timing race — device requested a lease before the router had
    committed the new reservation. -> `sudo ifdown <iface> && sudo
    ifup <iface>` (Debian 13 uses ifupdown, not dhclient) to force a
    fresh request. If it still doesn't match, and the mismatched IP
    sits below the DHCP pool's start address, it's functionally just
    as stable — update the reservation to match reality instead.
