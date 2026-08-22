# Configuration reference

`colors.yml` is the only file to edit. Keys are kebab-case and hold non-secret
values only. Credentials are `COLORS_PAR_<UPPER_SNAKE_KEY>` environment
variables in the gitignored `.envrc.private`.

## Required credentials

| Credential | Variable |
|---|---|
| Vultr API key | `COLORS_PAR_VULTR_API_KEY` |
| Cloudflare API token (zone-scoped) | `COLORS_PAR_CLOUDFLARE_API_TOKEN` |
| R2 state backend | `COLORS_PAR_R2_ACCESS_KEY_ID`, `COLORS_PAR_R2_SECRET_ACCESS_KEY` |
| GitHub token (read-only, public repos) | `COLORS_PAR_WAVEHOUSE_GITHUB_TOKEN` |

The GitHub token is stored on the server (mode 0600) for the backfill and the
poller; a fine-grained token with "Public repositories (read-only)" access and
no other permissions is sufficient. Anonymous access rate-limits at 60
requests/hour and cannot complete a backfill.

## Keys

| Key | Meaning |
|---|---|
| `profile` | Names the work directory, state keys, and cloud resources. Never overlay it. |
| `workdir` | Generated-output root, conventionally `.colors`. |
| `provider-compute` | Must be `vultr`. |
| `provider-dns` | Must be `cloudflare`. |
| `provider-backend` | `local`, `s3`, or `r2`. |
| `compute-prevent-destroy` | Keep `true`; guards `delete`. |
| `wavehouse-host` | Public dashboard hostname. Its registrable domain must be a Cloudflare zone; the package manages the proxied A record and Caddy obtains TLS. |
| `wavehouse-repo` | `owner/name` of the GitHub repository the dashboard tracks. |
| `wavehouse-poll-interval` | Seconds between GitHub Events/Actions polls. |
| `wavehouse-image` | WaveHouse gateway container image (tagged). |
| `wavehouse-clickhouse-image` | ClickHouse server image (tagged). |
| `wavehouse-caddy-image` | Caddy image (tagged). |
| `vultr-name` | Instance label (updates in place; never a hostname). |
| `vultr-region` / `vultr-plan` | Vultr region and plan. 2 vCPU / 4 GiB is the smallest comfortable class for ClickHouse plus the gateway. |
| `vultr-os-id` | Numeric OS id; 2284 is Ubuntu 24.04 LTS x64. |
| `vultr-ssh-keys` | Id of an SSH key already uploaded to the account. ForceNew: rotate by rebuilding. |
| `vultr-ssh-sources` / `vultr-http-sources` | CIDR allowlists for the firewall (22 and 80/443). |
| `r2-bucket` / `r2-endpoint` | State backend location (`r2` backend). |

## Data model

One wide `gh_events` table holds every event; 19 named pipes (registered with
`allowed_roles: ["public"]`) power the dashboard's charts; the browser is
anonymous and resolves to the read-only `public` role. Ships/publishes are
counted from releases plus successful runs of publish/release-named workflows
— the GitHub API cannot see registry-package webhook events, so this number
is a heuristic. Pushes are backfilled one row per commit; the poller records
one row per push event thereafter.

## Operations

```sh
ssh root@SERVER 'cd /opt/wavehouse && docker compose ps'
ssh root@SERVER 'journalctl -u wavehouse-poller --since -1h'
ssh root@SERVER 'systemctl start wavehouse-backfill'   # idempotent re-run
```

Changing `wavehouse-repo` and re-running `create` backfills the new repository
into the same table; drop the volumes first for a clean slate
(`docker compose down -v` on the server) if mixing repos is not wanted.
