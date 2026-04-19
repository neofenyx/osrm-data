# osrm-data

Pre-baked [OSRM](https://github.com/Project-OSRM/osrm-backend) graph artifacts for neofenyx dev environments.

Instead of every contributor running `osrm-extract` + `osrm-partition` + `osrm-customize` locally (14–18 GB RAM peak for Great Britain — too heavy for most laptops), this repo rebuilds the graph on GitHub Actions once a month and publishes the result as a tarball you can drop straight into your Docker volume.

## What's published

Each release contains a single tarball:

- `great-britain-latest.tar.xz` — graph files for `osrm-routed --algorithm mld` covering England, Scotland, Wales. Built from the Geofabrik `great-britain-latest.osm.pbf` snapshot using the `car.lua` profile.

The tarball contains the `.osrm` edge graph plus all `.osrm.*` MLD sidecar files — everything `osrm-routed` needs at runtime. The raw PBF is *not* included.

## Release channels

- **Dated tags** (`great-britain-latest-YYYY-MM-DD`) — immutable, safe to pin.
- **`latest` tag** — moving pointer to the most recent build. Default for consumers who always want the freshest data.

## Consumer usage

Point your OSRM data volume at a release asset. Requires `xz-utils` installed (pre-installed on most distros; on the OSRM Docker image it's `apt-get install xz-utils`):

```sh
OSRM_PREBUILT_URL=https://github.com/neofenyx/osrm-data/releases/download/latest/great-britain-latest.tar.xz
curl -L --fail "$OSRM_PREBUILT_URL" | tar -xJf - -C /data
osrm-routed --algorithm mld /data/great-britain-latest.osrm
```

This is how the `iris` repo's `osrm-prepare` container consumes the artifact — see `iris/etc/scripts/osrm-prepare.sh` for the fast-path.

## Rebuild cadence

- **Monthly**: automated rebuild on the 1st of each month at 03:00 UTC (`.github/workflows/release.yml`).
- **On demand**: trigger manually via `gh workflow run release.yml` or the Actions tab.

A standard `ubuntu-latest` runner (4 cores / 16 GB RAM + 16 GB swap we configure up-front) handles Great Britain in ~15–25 minutes.

## Pinned OSRM version

Graph files are version-tied to `ghcr.io/project-osrm/osrm-backend:v5.27.1` — the same image the consumer runs. If we bump OSRM in the consumer, bump it here at the same time or `osrm-routed` will refuse to load the file.

## Data licensing

Map data © OpenStreetMap contributors, available under the [Open Database License (ODbL)](https://opendatacommons.org/licenses/odbl/). Derived OSRM graph files published here inherit that license. See [openstreetmap.org/copyright](https://www.openstreetmap.org/copyright).
