# Podman Usage Guide

Build, run, and maintain the camofox-browser container with **Podman**.

All commands use the dedicated `Makefile.podman` (kept separate so the Docker
workflow is untouched). The image tag embeds the version and architecture:

```
camofox-browser:<VERSION>-<ARCH>     e.g. camofox-browser:135.0.1-x86_64
```

## Prerequisites

- Podman installed: `podman --version`
- `make` installed
- Run commands from the repository root

## Build

```bash
make -f Makefile.podman build
```

- Downloads Camoufox + yt-dlp **at build time** (first build: a few minutes).
- Version pins live at the top of `Makefile.podman` and must match the `ARG`s
  in the `Dockerfile`.
- Architecture is auto-detected (`x86_64` / `aarch64`). No flags needed.
- Subsequent builds are fast: podman reuses cached layers, so the download is
  skipped unless the pinned version changes.

## Start the container

```bash
make -f Makefile.podman up
```

- Builds the image first only if it doesn't already exist.
- Runs it as `camofox-browser` on port **9377**, with `--shm-size=2g` and
  `--restart unless-stopped` (survives host restarts).

Verify it's healthy:

```bash
make -f Makefile.podman ps     # container should show as Up
curl -s http://localhost:9377/health   # expect 200
```

## Kill / stop the container

```bash
make -f Makefile.podman down    # stop + remove the container
```

`down` is idempotent — safe to run when the container isn't running.

## Maintenance

| Task                          | Command                                   |
|-------------------------------|-------------------------------------------|
| Follow logs                   | `make -f Makefile.podman logs`            |
| Stop only (keep container)    | `make -f Makefile.podman stop`            |
| Restart (after code changes)  | `make -f Makefile.podman restart`         |
| Show container status         | `make -f Makefile.podman ps`              |
| Full teardown + rebuild       | `make -f Makefile.podman reset`           |

**Typical code-change loop:** edit code → `make -f Makefile.podman restart`
(the rebuild is quick thanks to layer caching).

**Upgrading the browser version:**

1. Bump `VERSION` / `RELEASE` at the top of `Makefile.podman`.
2. Update the matching `ARG CAMOUFOX_VERSION` / `ARG CAMOUFOX_RELEASE` in the
   `Dockerfile`.
3. `make -f Makefile.podman reset` — tears down the old container/image and
   builds the new one.

**Cleanup:**

```bash
podman rm -f camofox-browser        # force-remove a stuck container
podman image prune                  # remove dangling build layers
podman rmi camofox-browser:<TAG>    # remove a specific image
```

## Notes

- Rootless Podman is fine — port 9377 is non-privileged.
- This workflow does not use the `dist/` download cache from the Docker
  Makefile; binaries are fetched inside the image build.
- For boot-time auto-start (hosting), see the Quadlet example in the README's
  Docker section or run: `podman generate systemd --new --name camofox-browser`.
