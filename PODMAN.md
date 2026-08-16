# Podman Usage Guide

Build, run, and maintain the camofox-browser container with **Podman**.

All commands use the dedicated `Makefile.podman` (kept separate so the Docker
workflow is untouched). The image tag embeds the version and architecture:

```
camofox-browser:<VERSION>-<ARCH>     e.g. camofox-browser:135.0.1-x86_64
```

## Quick Start

```bash
# Build and start (auto-detects arch: aarch64 on ARM, x86_64 on Intel)
make -f Makefile.podman up

# Stop and remove the container
make -f Makefile.podman down

# Force a clean rebuild (e.g. after upgrading VERSION/RELEASE)
make -f Makefile.podman reset

# Verify health
make -f Makefile.podman health
```

## Build

```bash
make -f Makefile.podman build
```

- Downloads Camoufox + yt-dlp **at build time** (first build: a few minutes).
- Version pins live at the top of `Makefile.podman` and must match the `ARG`s
  in the `Dockerfile`.
- Architecture is auto-detected (`x86_64` / `aarch64`). No flags needed.
- Subsequent builds are fast: Podman reuses cached layers, so the download is
  skipped unless the pinned version changes.

## Maintenance

| Task | Command |
|------|---------|
| Follow logs | `make -f Makefile.podman logs` |
| Stop only (keep container) | `make -f Makefile.podman stop` |
| Restart (after code changes) | `make -f Makefile.podman restart` |
| Show container status | `make -f Makefile.podman ps` |
| Full teardown + rebuild | `make -f Makefile.podman reset` |

**Typical code-change loop:** edit code → `make -f Makefile.podman restart`
(the rebuild is quick thanks to layer caching).

## Configuration

### API Key (Cookie Import)

Create an env file and pass it via `ENV_FILE`:

```bash
echo 'CAMOFOX_API_KEY=your-generated-key' > ~/.camofox/camofox.env
make -f Makefile.podman down
make -f Makefile.podman up ENV_FILE=~/.camofox/camofox.env
```

### Extra Flags

Pass additional `podman run` flags via `EXTRA_FLAGS`:

```bash
make -f Makefile.podman up EXTRA_FLAGS="--network host"
```

## Troubleshooting

```bash
podman rm -f camofox-browser        # force-remove a stuck container
podman image prune                  # remove dangling build layers
podman rmi camofox-browser:<TAG>    # remove a specific image
```
