---
name: package-wavehouse-green
description: Provisions a public WaveHouse analytics demo — ClickHouse, the WaveHouse gateway, and the live GitHub stats dashboard — on one Vultr instance.
license: MIT
---

# WaveHouse demo with Green

Operate one WaveHouse demo deployment from non-secret `colors.yml`. Read
[references/configuration.md](references/configuration.md) before changing
configuration or running a lifecycle operation.

The demo serves the WaveHouse stats dashboard (the upstream project's own
dogfooding page) for the configured GitHub repository: history is backfilled
from the GitHub API at create time and a 60-second poller keeps the live feed
streaming over SSE.

## Safety

- Keep credentials in gitignored `.envrc.private` as `COLORS_PAR_*` variables.
- Never set `COLORS_PAR_PROFILE` or edit/commit `.colors/`.
- Keep `compute-prevent-destroy: true`; deletion requires separate explicit
  authorization and a one-run environment override.
- Build and dry-run before a real create.
- Only Caddy 80/443 and key-only SSH are public; the gateway's operator key is
  generated on the server and never leaves it.

```sh
./green build
./green create --dry-run
./green create
```

Real create ends with public HTTPS health, dashboard, and backfilled-data
acceptance checks.

Compute and SSH ownership are delegated to `colors-compute`. Omitted provider SSH settings use `~/.ssh/<profile>`; supplied account key references remain external. The package maintains the profile SSH alias with a locked updater and refuses foreign stanzas. R2/S3 compute states live under `<profile>/compute/`; an existing `<profile>/wavehouse-infrastructure.tfstate` requires reviewed migration before normal lifecycle commands.
