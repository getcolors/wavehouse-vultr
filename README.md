# wavehouse-vultr

Desired state for the public WaveHouse analytics demo at
**https://stats.bigconfig.space** — the live GitHub stats dashboard for
[Wave-RF/WaveHouse](https://github.com/Wave-RF/WaveHouse), computed by a
self-hosted [WaveHouse](https://wavehouse.dev) + ClickHouse stack on one
Vultr instance. Provisioned by the
[wavehouse Package Skill](https://github.com/getcolors/wavehouse).

## Layout

- `colors.yml` — the only editable desired state (host, tracked repo, poll
  interval, images, Vultr and state-backend boundary).
- `.envrc.private` (gitignored) — `COLORS_PAR_VULTR_API_KEY`,
  `COLORS_PAR_CLOUDFLARE_API_TOKEN`, `COLORS_PAR_R2_ACCESS_KEY_ID`/`SECRET`,
  `COLORS_PAR_WAVEHOUSE_GITHUB_TOKEN`.
- `.ssh/` (gitignored) — the deployment's disposable SSH keypair, held by a
  per-deployment agent that `.envrc` starts; the public half is the
  `wavehouse-vultr-disposable` key in the Vultr account.
- `.agents/skills/package-wavehouse-green/` — installed Package Skill payload;
  the root `./green` is a copy of its launcher.

## Lifecycle

```sh
direnv allow          # once; builds the toolchain, starts the SSH agent
./green build         # render only, credential-free
./green create --dry-run
./green create        # converge; ends with HTTPS + data acceptance checks
```

Changing the tracked repository is a one-key edit (`wavehouse-repo`) followed
by `./green create`; the backfill loads the new history idempotently.

## Operations

```sh
ssh root@SERVER 'cd /opt/wavehouse && docker compose ps'
ssh root@SERVER 'journalctl -u wavehouse-poller --since -1h'
ssh root@SERVER 'systemctl start wavehouse-backfill'   # idempotent re-run
```

The stack is a demo: state lives in Docker volumes on the instance, with no
backups — rebuild by `delete` (separately authorized) and `create`.
