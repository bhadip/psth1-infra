# Architecture — prasanti.com Web

## Data flow
1. Cloudflare is authoritative DNS for prasanti.com (migrated from
   idwebhost nameservers; email/MX untouched throughout).
2. `cloudflared` container makes an outbound-only connection to
   Cloudflare's edge (psth1-tunnel) — no inbound ports opened on the
   router at all.
3. Cloudflare routes public requests for each subdomain back down the
   tunnel to `caddy:80` inside the private Docker network.
4. Caddy (`auto_https off`, plain :80 listener — TLS is terminated at
   Cloudflare's edge, not here) host-matches each subdomain and
   serves/redirects accordingly.

## Services & boot chain
| What            | Mechanism                              | Auto-start |
|------------------|-----------------------------------------|------------|
| prasanti-caddy   | docker compose, restart:unless-stopped  | yes        |
| prasanti-tunnel  | docker compose, restart:unless-stopped  | yes        |

## Routes (all confirmed live)
| Host                    | Behavior                              |
|-------------------------|----------------------------------------|
| www.prasanti.com        | static "Under Construction" page      |
| exaguard.prasanti.com   | redirect -> YouTube                   |
| bambang.prasanti.com    | redirect -> LinkedIn                  |
| gita.prasanti.com       | redirect -> LinkedIn                  |
| aisha.prasanti.com      | redirect -> LinkedIn                  |
| prasanti.com (apex)     | Cloudflare Redirect Rule -> www        |
| http:// (any host)      | Cloudflare "Always Use HTTPS" -> https |

## Key design decisions
- Cloudflare Tunnel, not port-forwarding — psth1's real IP is never
  exposed; only Cloudflare's edge is public-facing.
- `auto_https off` in Caddy — Caddy previously tried to ALSO obtain
  its own Let's Encrypt certs and failed (ACME challenge can't reach
  a tunnel-only origin). TLS belongs to Cloudflare's edge only.
- Apex redirect via Cloudflare Redirect Rule (custom filter expression
  `http.host eq "prasanti.com"`), not routed through Caddy — simpler,
  and the apex never needs to touch the tunnel/origin at all.
- SPF (`v=spf1 include:_spf.google.com ~all`) and DMARC
  (`p=none`, Cloudflare-managed reporting) added after the DNS
  migration — monitoring mode, not enforcing yet.

## Known limitations
- DKIM not configured (needs Google Workspace Admin console signing
  setup, then a DNS TXT record here).
- DMARC still `p=none` — tighten to `p=quarantine` only after a
  couple of weeks of clean reports in Cloudflare's dashboard.
