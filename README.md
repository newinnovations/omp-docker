# omp-docker

Automatically builds the [oh-my-pi](https://github.com/can1357/oh-my-pi) Docker image
from upstream's own `Dockerfile` and publishes it as `<version>` (leading `v` stripped,
e.g. `v18.8.6` → `18.8.6`) and `latest`.

## How it works

`.github/workflows/build-omp.yml` runs every hour (and on demand):

1. **check** – finds the highest stable upstream tag (`vX.Y.Z`) via `git ls-remote`,
   then checks whether `<version>` already exists on Docker Hub and GHCR.
   The registries are the "already processed" state — nothing is committed to this repo.
2. **build** – only if the tag is missing from either registry: checks out upstream at
   that tag, builds once with Buildx (`linux/amd64`, GitHub Actions layer cache) and
   pushes to both.

A failed build doesn't publish anything, so the next hourly run retries automatically.
Runs never overlap (concurrency group).

## Manual runs

Actions → *Build & publish oh-my-pi image* → *Run workflow*:

- `tag`: build a specific upstream tag (e.g. `v18.8.5`); empty = latest.
- `force`: rebuild/overwrite even if the tag already exists in both registries.

Note: building an older tag also moves `latest` to it.

## Notes / caveats

- **Scheduled workflows are auto-disabled by GitHub after 60 days without repo
  activity.** Re-enable under Actions if that happens (or push a commit occasionally).
- Schedule times are best-effort; GitHub may delay hourly runs.
- Only `linux/amd64` is built. The image compiles a Rust native addon, so arm64 via QEMU
  would be very slow; add `linux/arm64` to `platforms` if you need it (or use an arm runner).
- Builds take a while (Rust compile); the job timeout is 4 hours.
