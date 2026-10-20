# Dashboard (homepage)

A self-hosted [gethomepage/homepage](https://github.com/gethomepage/homepage) dashboard
that acts as the single landing page for a homelab box. It shows service tiles with
live Docker container status, host CPU / RAM / disk / temperature, a search bar,
bookmarks, RSS news tiles, and per-service widgets, under a custom Fallout Pip-Boy theme.

<img width="1348" height="1105" alt="image" src="https://github.com/user-attachments/assets/e55c19b5-3a66-4761-8175-f7c17e147520" />


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
`.env` (gitignored) supplies four values that are injected into the config at runtime,
so no real IPs or secrets live in the committed files:

| Var | Used for |
|---|---|
| `HOST_IP` | the lab box's LAN IP: tile links, `startUrl`, and `HOMEPAGE_ALLOWED_HOSTS` |
| `GATEWAY_IP` | the router / gateway tile |
| `PIHOLE_KEY` | the Pi-hole widget password |
| `JELLYFIN_KEY` | Jellyfin API key for the Media tile widget (create it in Jellyfin after its first-run wizard; empty is fine until then) |

In the YAML these appear as `{{HOMEPAGE_VAR_HOST_IP}}`, `{{HOMEPAGE_VAR_GATEWAY_IP}}`,
`{{HOMEPAGE_VAR_PIHOLE_KEY}}` and `{{HOMEPAGE_VAR_JELLYFIN_KEY}}`. Homepage substitutes them from the matching
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
- `services.yaml` — service tiles, grouped **Lab / Network / AI / Projects / Infra /
  Media / News**. `server: local` + `container:` pulls live Docker status;
  `siteMonitor` adds an up/down check; some tiles carry widgets (Pi-hole stats, Ollama
  default model, Jellyfin library counts + now playing). **News** tiles use the built-in
  `feed` widget: one RSS/Atom `url` per tile, fetched server-side and cached 10 min;
  swap sources by editing `url`/`href`.
- `widgets.yaml` — header widgets: host resources, search, clock. The stock greeting
  and logo are commented out because the Pip-Boy banner replaces them.
- `bookmarks.yaml` — the link bar (Security, Recon, Docs).
- `settings.yaml` — title, theme (`dark` + `emerald`), layout and column counts.
- `docker.yaml` — the Docker socket definition.
- `custom.css` — the Pip-Boy skin: phosphor-green on near-black, CRT scanlines and
  flicker, VT323 / Share Tech Mono fonts, monochrome-tinted icons, and the full-width
  header banner injected from `images/pip-banner.png` (served via the `./images`
  bind mount in `docker-compose.yml`). Homepage reads `custom.css` at startup, so
  `docker compose restart` after editing it. Delete it for the stock look; the
  `--pip` variable at the top switches the accent colour.

## Notes
- **Monitors use `host.docker.internal`** (the host gateway), not the LAN IP, so checks
  keep working if the box's address changes or the stack is migrated.
- **Loopback-only services** (bound to 127.0.0.1) are unreachable from inside the
  container and therefore have no `siteMonitor`.
- Any new hostname you browse to must be in `HOMEPAGE_ALLOWED_HOSTS` or Homepage
  rejects the request.
- **Jellyfin widget needs `version: 2`** (already set) for Jellyfin 12 or newer, which
  dropped the legacy `/emby/` API prefix the v1 widget uses.
- Migrating: set `HOST_IP` in `.env` and `docker compose up -d` (env is baked in at
  container create, so a plain restart is not enough). Bring up DashboardStack first so
  the `*.lab` DNS and Caddy routes exist.
- Icons: any name from [dashboard-icons](https://github.com/homarr-labs/dashboard-icons)
  (`foo.png`) or `mdi-*` / `si-*`. Widgets: https://gethomepage.dev/widgets/

## Related
- Companion services: https://github.com/dsgregory07/DashboardStack
