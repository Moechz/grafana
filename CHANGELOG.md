# Changelog

All notable user-visible changes to the Grafana TOS application package.

## 13.3.0-1 — 2026-11-11

### Added
- First release. Grafana (`grafana/grafana:13.3.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8067 and running
  as a non-root user (472:472).
- Persistent data is kept under `/Volume1/DockerAppData/grafanadocker/`.
