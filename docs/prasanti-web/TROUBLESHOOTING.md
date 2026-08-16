# Troubleshooting — prasanti.com Web

## Cheat sheet
Containers: prasanti-caddy, prasanti-tunnel
Logs: `docker compose logs caddy|cloudflared --tail 30`

## Symptom -> Cause -> Fix
1. Caddy logs full of ACME/Let's Encrypt "challenge failed" errors
   Caddy trying to obtain its own cert — impossible behind a tunnel.
   -> confirm `auto_https off` is set in the Caddyfile global block.

2. New subdomain returns 522 (connection timed out)
   A leftover A record for that hostname exists in Cloudflare DNS,
   conflicting with/blocking the Tunnel's own CNAME record.
   -> DNS -> Records -> delete the old A record for that host -> re-add
   the hostname route in the Tunnel dashboard.

3. www.prasanti.com loads the WRONG subdomain's content
   Browser cached an old 301 redirect from before the routing was
   fixed. Not a server-side bug. -> test in an incognito window.

4. Apex (prasanti.com, no www) times out
   Redirect Rule match condition wrong — "Wildcard pattern" requires
   a full URL with protocol, or a "URI Full" match doesn't equal a
   bare hostname. -> use Custom filter expression,
   `Hostname equals prasanti.com`, not URI Full / Wildcard pattern.

5. http:// doesn't redirect to https://
   -> SSL/TLS -> Edge Certificates -> "Always Use HTTPS" -> On.

6. Email stops arriving after any DNS change here
   STOP further changes immediately. Re-check MX records against the
   documented baseline (see RUNBOOK.md step 1). This project treats
   email delivery as higher priority than any web routing change.
