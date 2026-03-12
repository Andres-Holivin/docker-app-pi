# Self-Hosted Sentry

This folder vendors the official `getsentry/self-hosted` release `26.2.1` and pins the public bind port to `9001` to avoid the existing Portainer mapping on `9000`.

Upstream documentation: https://develop.sentry.dev/self-hosted/

## First Run

```bash
cd /home/andres/docker-app-pi/sentry
./install.sh
docker compose up -d
```

Open Sentry at `http://localhost:9001`.

## Daily Commands

```bash
cd /home/andres/docker-app-pi/sentry
docker compose up -d
docker compose down
docker compose logs -f web worker relay nginx
```

## Admin User

Create the first admin user after the stack is up:

```bash
cd /home/andres/docker-app-pi/sentry
docker compose run --rm web createuser
```

## Local Overrides

- `.env` is tracked and pins the workspace defaults.
- `.env.custom` is ignored and can override `.env` on this host.
- `sentry/sentry.conf.py`, `sentry/config.yml`, and `symbolicator/config.yml` are created from example files by `./install.sh` and are intentionally ignored.

See `.env.custom.example` for the supported repo-local overrides.

## Notes

- The official self-hosted stack is resource-heavy. Upstream guidance is roughly `4 CPU`, `16 GB RAM + 16 GB swap`, and `20 GB` free disk.
- This import keeps the upstream named-volume layout for compatibility with the official installer instead of rewriting storage to local bind mounts.
- SMTP is left unset by default. Add a real relay later through `.env.custom` when you want outbound email.
