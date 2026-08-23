# MindsHub (psth1)

Self-hosted MindsHub Cowork on psth1, published at `mindshub.prasanti.com`.
Protected by Cloudflare Access: Google login only, allowlisted emails only.

## Architecture

Internet
  -> Cloudflare edge (SSL + Access policy)
  -> psth1-tunnel (cloudflared container, remotely managed via token)
  -> http://192.168.50.2:8081
  -> mindshub-web (nginx) -> mindshub-api (cowork-server)

## Files on psth1

- App compose: `~/minds/docker-compose.minds.yml` (reference copy in `config/mindshub/`)
- Data: docker volume `minds_cowork-data` (SQLite DB, settings, model API keys)
- Tunnel: container `prasanti-tunnel` (auto-restart; config pulled from Zero Trust)

## Cloudflare configuration (Zero Trust dashboard)

### Tunnel route (Networks > Tunnels > psth1-tunnel > Published application routes)
| Hostname              | Service                  |
|-----------------------|--------------------------|
| mindshub.prasanti.com | http://192.168.50.2:8081 |

### Access (Access > Applications > mindshub)
- Domain: `mindshub.prasanti.com`
- Login methods: **Google only** (Cloudflare OneAuth disabled)
- Policy `Allow Google Users`: Emails is `bhadip@gmail.com`
  (add more emails here to grant access; everyone else is denied by default)
- Purpose justification: OFF

### Google OAuth (Google Cloud Console > Credentials)
Authorized redirect URI:
`https://delicate-brook-2c40.cloudflareaccess.com/cdn-cgi/access/callback`

## Operations

    # restart / rebuild
    cd ~/minds && docker compose -f docker-compose.minds.yml up -d --build

    # logs
    docker logs -f mindshub-web
    docker logs -f mindshub-api

    # grant / revoke user access
    Zero Trust > Access > Applications > mindshub > Policies > edit emails

## Migration to new hardware

1. Install Docker; clone this repo.
2. `mkdir -p ~/minds && cp config/mindshub/docker-compose.minds.yml ~/minds/`
   (adjust host IP in the tunnel route if LAN IP changed).
3. `cd ~/minds && docker compose up -d`; re-enter model API keys in the web UI,
   or restore the `cowork-data` volume from backup.
4. Zero Trust:
   - run cloudflared with the tunnel token (see docs/networking)
   - published application route: mindshub.prasanti.com -> http://<new-ip>:8081
   - verify Access application + policy emails
5. Test in incognito: Google login -> straight into MindsHub.

## Decisions & gotchas (2026-08-22)

- **No custom Telegram approval gatekeeper.** A FastAPI proxy with Telegram
  approve/reject buttons was prototyped and abandoned (webhook callback
  handling proved unreliable). Native Cloudflare Access policies are simpler
  and fully reliable.
- **Keep hostnames second-level.** Cloudflare universal SSL only covers
  `*.prasanti.com`; `webhook.mindshub.prasanti.com` failed with TLS
  handshake errors.
- **cloudflared runs in a container**: `localhost` points inside the tunnel
  container, not the host. Always route to the host LAN IP (192.168.50.2).
- **Port 8080 was already in use** on psth1; web is published on 8081.
