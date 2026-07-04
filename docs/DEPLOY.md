# Deploy with Piper

This is a showcase payload — you deploy *it* with [Piper](https://github.com/getpiper/piper),
the open-source PaaS that gives you `git push → live HTTPS URL` on hardware you own. This
doc is the canonical recipe; the README has the short version.

## Prerequisites

1. **`piperd` running on your box.** Build and start it from the
   [Piper repo](https://github.com/getpiper/piper). Defaults: control API on
   `127.0.0.1:8088`, managed Caddy on `:443`/`:80`, app container port `8080`, base domain
   `piper.localhost`.
2. **Docker** on the same box (Piper builds and runs your `Dockerfile`).
3. **`piper` CLI** installed, talking to `piperd` (default `PIPERD=http://127.0.0.1:8088`).

## Deploy (LAN-only, Plan 1)

From this repo:

```bash
piper create sample-app
piper deploy sample-app --path .
```

Piper builds the multi-stage `Dockerfile`, runs the container (port `8080`, bound
`0.0.0.0`, so its managed Caddy can reach it), TCP-health-checks the published port, and
routes `sample-app.piper.localhost` to it. Then:

```bash
curl http://sample-app.piper.localhost/         # home SSR
curl http://sample-app.piper.localhost/about     # about SSR
curl http://sample-app.piper.localhost/health    # -> ok
```

Add `piper.localhost` to your machine's DNS or use `--resolve` if needed.

## Public HTTPS (Plan 2 — relay + DNS-01 TLS)

Once you've enrolled your agent with a `piper-relay` (self-hosted or the hosted
convenience instance) and configured your domain + DNS credentials, Piper terminates TLS
on-box with a lego-issued wildcard cert and tunnels public `:443` traffic from the relay
to your box without exposing your network:

```bash
curl https://sample-app.<your-domain>/
```

See the Piper repo's Plan 2 docs for enrollment and DNS setup.

## Local smoke (optional, no Piper)

```bash
make build
docker build -t sample-app:dev . && docker run --rm -p 8080:8080 sample-app:dev
curl localhost:8080/health   # -> ok
```

This proves the `Dockerfile` + standalone `server.js` contract (one port, `$PORT` default
`8080`, `HOSTNAME=0.0.0.0`, `ok` on `/health`) that Piper depends on — without needing
`piperd` running.