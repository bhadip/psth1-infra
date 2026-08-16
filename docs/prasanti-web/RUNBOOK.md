# Runbook — prasanti.com Web

## Add/change a subdomain redirect
    cd /home/tom/docker/prasanti-site
    nano Caddyfile          # add a new `@name host x.prasanti.com { ... }`
                             # handle block, matching existing entries
    docker compose restart caddy

Then in Cloudflare Zero Trust -> Networks -> Tunnels -> psth1-tunnel ->
Published application routes -> add `x.prasanti.com` -> `http://caddy:80`.
If the hostname previously had an A record, delete it first (Cloudflare
won't create the Tunnel/CNAME record while a conflicting A record exists).

## Edit the "Under Construction" page
    /home/tom/docker/prasanti-site/site/index.html
Edit only the `:root { --headline / --tagline / --eta-label / --eta-value }`
block near the top — no other markup/CSS needs touching.

## Check status
    docker ps                                  # both containers Up?
    docker compose logs caddy --tail 30
    docker compose logs cloudflared --tail 30  # look for
                                                 # "Registered tunnel connection"

## DNS / email safety check (before ANY future DNS change)
1. Document current MX records first.
2. Make the change.
3. Verify MX records identical afterward (mxtoolbox.com or `dig MX`).
4. Send yourself a real test email before considering it done.
