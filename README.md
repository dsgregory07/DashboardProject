# Dashboard (homepage)

A self-hosted [gethomepage/homepage](https://github.com/gethomepage/homepage) dashboard
that acts as the single landing page for a homelab box. It shows service tiles with
live Docker container status, host CPU / RAM / disk / temperature, a search bar,
bookmarks, and per-service widgets, under a custom terminal/neon theme.

It pairs with the companion stack
[DashboardStack](https://github.com/dsgregory07/DashboardStack), which provides the
monitoring, logs, DNS, and reverse-proxy services several tiles point at.

## Requirements
- Docker + Docker Compose.
- Read access to the Docker socket (the container runs as the `docker` group so it can
  read `/var/run/docker.sock`, mounted read-only).

## Setup
```
cp .env.example .env      # then edit .env with your values
docker compose up -d
```
`.env` (gitignored) supplies three values that are injected into the config at runtime,
so no real IPs or secrets live in the committed files:

| Var | Used for |
|---|---|
| `HOST_IP` | the lab box's LAN IP: tile links, `startUrl`, and `HOMEPAGE_ALLOWED_HOSTS` |
| `GATEWAY_IP` | the router / gateway tile |
| `PIHOLE_KEY` | the Pi-hole widget password |

In the YAML these appear as `{{HOMEPAGE_VAR_HOST_IP}}`, `{{HOMEPAGE_VAR_GATEWAY_IP}}`,
and `{{HOMEPAGE_VAR_PIHOLE_KEY}}`. Homepage substitutes them from the matching
`HOMEPAGE_VAR_*` environment variables set in `docker-compose.yml`, and
`HOMEPAGE_ALLOWED_HOSTS` reads `${HOST_IP}` directly.

## Running / updating
```
docker compose logs -f                        # watch
docker compose pull && docker compose up -d    # update to the latest image
```
Edits under `config/` are picked up live; only `.env` or `docker-compose.yml` changes
need a `docker compose up -d`.

## Config layout (`config/`)
- `services.yaml` — service tiles, grouped **Lab / Network / AI / Projects / Infra**.
  `server: local` + `container:` pulls live Docker status; `siteMonitor` adds an
  up/down check; some tiles carry widgets (Pi-hole stats, Ollama default model).
- `widgets.yaml` — header widgets: greeting, logo, host resources, search, clock.
- `bookmarks.yaml` — the link bar (Security, Recon, Docs).
- `settings.yaml` — title, theme (`dark` + `emerald`), layout and column counts.
- `docker.yaml` — the Docker socket definition.
- `custom.css` — the terminal/neon skin (near-black background, JetBrains Mono,
  neon-green cards that glow on hover). Delete it to return to the stock look.

## Notes
- **Monitors use `host.docker.internal`** (the host gateway), not the LAN IP, so checks
  keep working if the box's address changes or the stack is migrated.
- **Loopback-only services** (bound to 127.0.0.1) are unreachable from inside the
  container and therefore have no `siteMonitor`.
- Any new hostname you browse to must be in `HOMEPAGE_ALLOWED_HOSTS` or Homepage
  rejects the request.
- Icons: any name from [dashboard-icons](https://github.com/homarr-labs/dashboard-icons)
  (`foo.png`) or `mdi-*` / `si-*`. Widgets: https://gethomepage.dev/widgets/

## Related
- Companion services: https://github.com/dsgregory07/DashboardStack
