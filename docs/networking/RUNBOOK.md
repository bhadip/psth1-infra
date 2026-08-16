# Runbook — Networking

## Wake psth1 remotely (from same LAN segment)
    wakeonlan -i 192.168.50.255 b8:ca:3a:d8:e5:57      # from a Mac/Linux
                                                          # device on 192.168.50.x
    ether-wake -b -i br0 b8:ca:3a:d8:e5:57              # from AX3000 via SSH

## Wake psth1 from truly outside the house
Not yet solved — needs the Raspberry Pi (independent Tailscale node
on 192.168.50.x). See ARCHITECTURE.md. Until then: no remote-wake path
exists once psth1 is fully powered off.

## Check AdGuard filtering
    https://psth1.gila-karat.ts.net:3000     # dashboard, Tailscale only
Toggle WiFi off/on on a test device to force a fresh DHCP request
before checking query logs.

## Add a custom block/allow rule (durable, editable from anywhere)
Edit `prasanti.block.list` in the GitHub repo directly (web or mobile
app) — AdGuard re-fetches the raw URL automatically, no server access
needed. For one-off manual rules: Filters -> Custom filtering rules
(AdBlock syntax: `||domain.com^` block, `@@||domain.com^` allow).
NOTE: domain-level only — cannot block/allow by raw IP or URL path.

## Access AX3000 router securely from outside
    ssh prasanti@192.168.50.1      # while Tailscale connected + subnet
                                     # route 192.168.50.0/24 approved
                                     # in the admin console
Requires psth1 to be ON (it's the subnet-routing gateway). Does not
work if psth1 is off — see Wake-on-LAN limitation above.

## Pull + read the router's log
    scp prasanti@192.168.50.1:/tmp/syslog.log ~/Downloads/
    lnav /home/tom/ax3000-logs/*.log     # live filter: press /
Auto-pull runs every 6h via systemd timer `ax3000-log-pull.timer`,
30-day auto-cleanup.

## Force psth1's DNS to persist after any network change
1. Update `docker-compose.yml` (adguard) port bindings to the new IP.
2. `docker compose up -d`
3. AX3000 -> LAN -> DHCP Server -> DNS Server 1 -> update to match.
