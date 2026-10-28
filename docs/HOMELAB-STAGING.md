# ThiepCuoi — Homelab staging (not production)

Scope: private family use, local-only test, no Cloudflare Tunnel, no change to existing NPM. This configuration is a **compatibility smoke test**, not a security endorsement of the encoded PHP application.

## Prerequisites

- Docker Engine and Compose v2+
- Free host TCP port 18686
- The full repo cloned into /srv/docker-data/thiepcuoi/src

## Start (manual, opt-in)

```bash
cd /srv/docker-data/thiepcuoi/src
docker compose -f compose.staging.yaml config -q
docker compose -f compose.staging.yaml build
docker compose -f compose.staging.yaml up -d
docker compose -f compose.staging.yaml ps
curl -I http://127.0.0.1:18686/
```

Access from the host only: http://127.0.0.1:18686/ . If opening remotely, use an SSH local port forward (`ssh -L 18686:127.0.0.1:18686 user@homelab`) or an equivalent existing trusted access method. Do not publish directly to the public Internet.

### Important constraints

- Source includes encrypted PHP payloads executed via eval(). These have not been fully audited.
- Docker is not a perfect sandbox. For first execution of untrusted code, a dedicated VM isolated from household and company systems is safer than this shared host. No guarantee is made that the app does not make outbound connections; this Compose file does **not** implement a reliable egress block.
- The PHP built-in server is intended only for compatibility/staging, not a production front end.
- Tunnel / update utilities must not be run from the original repo. This Compose command runs PHP directly without the provided launchers.
- Container is not attached to the existing `npm` network. Domain setup is deferred.
- Stateful content lives in dedicated named volumes with Compose project prefix `thiepcuoi-staging`.
- Initial image compatibility and functional behavior have not been tested on the target host.

## Stop without deleting data

```bash
docker compose -f compose.staging.yaml down
```

Do NOT use `down -v`: it removes named volumes holding the database and uploaded media.

## Update procedure (after reviewed PR merge)

```bash
cd /srv/docker-data/thiepcuoi/src
git fetch origin
git status --short
# Review and pin the intended commit before checkout/pull.
docker compose -f compose.staging.yaml build
docker compose -f compose.staging.yaml up -d
```

Backup volumes and test restore before deploying updates to real family data.
