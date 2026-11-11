# grafana-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Grafana](https://github.com/grafana/grafana).

## What this is

A four-file TOS Docker application archive (`grafanadocker.tar.gz`) that installs
Grafana on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `grafanadocker`)
- `grafanadocker.lang` — 23-language superset (TOS requires 14)
- `grafanadocker.svg` — application icon (3898 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `grafana/grafana:13.3.0`

## How it works

| | |
|---|---|
| Container image | `grafana/grafana:13.3.0` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8067/` (container port 3000) |
| Runs as | non-root, `user: "472:472"` |
| Persistent data | `/var/lib/grafana` → `/Volume1/DockerAppData/grafanadocker/var` |
| Logs | `docker logs grafanadocker` (stdout/stderr) |



## Design highlights

- **Non-root.** `user: "472:472"`; verified on real hardware (the container reports uid 472).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 3000.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/grafanadocker/var` | Grafana application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/grafanadocker.tar.gz + .sha256
scripts/build.sh 13.3.0-2   # packaging iteration bump
scripts/build.sh 13.3.0-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/grafanadocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v13.3.0-1`**, assets `grafanadocker.tar.gz` +
   `grafanadocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `grafanadocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `13.3.0-1` → automated validation → review.

## License / branding

Grafana is distributed by its upstream authors under **AGPL-3.0**. The container image is the official image from Docker Hub (`grafana/grafana:13.3.0`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- The application runs inside a container; data lives in the application data directory (`/Volume1/DockerAppData/grafanadocker/`) as described above.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
