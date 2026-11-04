# Self-hosted deployment (gvansh24 fork)

Everything fork-specific lives in `deploy/` and `.github/workflows/deploy-oracle.yml`
on the **`live`** branch, so syncing with upstream never conflicts.

| | |
|---|---|
| URL | http://92.4.88.247:8080/ |
| Host | openclaw OCI instance, `~/server/gods-eye-view` (branch `live`) |
| Container | `gods_eye_view` on docker network `web-gateway`, port 4173 (not published) |
| Gateway | `master_nginx` → `servers/gods-eye-view.conf` in gvansh24/server-infra listens on :8080 |
| Serve mode | `vite build` + `vite preview` (all `/api` providers; no dev key-setup panel) |

## Branches

- `main` — pure mirror of `bilawalsidhu/gods-eye-view`. Update with GitHub's **Sync fork**
  or `git fetch upstream && git push origin upstream/main:main`.
- `live` — `main` + the deploy files. Pushing `live` deploys (GitHub Actions).

Promote upstream changes:

```bash
git fetch upstream
git switch live
git merge upstream/main
git push origin live
```

## First-time server setup

```bash
cd ~/server
git clone -b live https://github.com/gvansh24/gods-eye-view.git
cd gods-eye-view
cp deploy/env.example .env && chmod 600 .env   # keys are optional; see the comments in it
sudo docker compose -f deploy/docker-compose.yml --env-file .env up -d --build
```

The OCI security list must allow ingress TCP 8080. Host iptables needs no change:
Docker's FORWARD rules handle published ports.

## Why not upstream's Dockerfile

Upstream's root `Dockerfile` and `compose.yaml` run the dev server and trust the
container gateway for Provider Settings, which is only safe on loopback. `deploy/Dockerfile`
serves the production build with `vite preview`, where that panel is not registered.

## Why the gateway headers matter

`servers/gods-eye-view.conf` must send `Host $http_host` (with the port) or the same-site
gate returns 403 on terrain. The forwarding headers make upstream's gate refuse routes
that spend keys (Places, TomTom, OpenAI, Gemini), so those features stay off here. The
deploy workflow's smoke test fails if either behaviour changes.

## Why a separate port

The app hard-codes root paths (`/api/...`, `/models/...`, `/logo.svg`), so it must be
served at `/`. When a domain exists, change `listen`/`server_name` in the gateway's
`servers/gods-eye-view.conf` and add the name to `GEV_ALLOWED_HOSTS` in `.env`.
