# Sync cadence for this fork

This fork (`t4coffee/my-cloudflare-os-starter`) derives from `cloudflare/cloudflare-os-starter`, which in turn pins a `cloudflare-os` git submodule at a reviewed commit. Two separate things get out of date, on two different rhythms.

## Remotes

- `upstream` → `https://github.com/cloudflare/cloudflare-os-starter.git` — fetch only, never push.
- `personal` → `https://github.com/t4coffee/my-cloudflare-os-starter.git` — push target for `main`.
- `.gitmodules` → `cloudflare/cloudflare-os`, pinned; do not repoint it at a personal fork.

## Cadence

| What | How often | Why |
| --- | --- | --- |
| Starter `main` (wrapper code, `packages/cf-os`, docs) | **Weekly**, read-only check; merge monthly | Cheap to check, low churn. Merge before it grows stale. |
| Upstream `cloudflare-os` pin | **Only with a starter bump**, or monthly review of upstream releases | Cloudflare OS is early-access: upstream pin moves are reviewed deliberately, not tracked continuously. |
| Deployment (`pnpm cf-os check`) | Before every deploy | Validates `deployment.jsonc` against the account. |

## Weekly check (read-only)

```sh
git fetch upstream
git log --oneline HEAD..upstream/main          # starter changes since your fork
git -C cloudflare-os log --oneline -1          # which upstream commit you are pinned to
```

If nothing relevant changed, stop. Nothing to merge.

## Merging starter updates

```sh
git fetch upstream
git merge upstream/main                        # resolve conflicts; deployment.jsonc is yours
git submodule update --init --recursive        # follow the pin the new starter declares
pnpm install
pnpm cf-os check                               # dry-run build + deploy of every Worker
```

The supported path is `npx cf-os@latest update`, which merges starter `main`, moves the reviewed pin, and writes `.cf-os/update.md` for your coding agent. Use the manual commands above only when you want to inspect each step.

## Upgrade checklist before moving the pin past the starter's

From `docs/customization.md#upgrade`:

1. Read the upstream release notes and docs for behavior changes.
2. Deploy to a non-production instance or `workers.dev` route first.
3. Verify sign-in, `/admin`, Context, Scheduler, Web Search, and any custom Gatekeeper.
4. Move the pin only after the new release is verified; roll back with `wrangler rollback` or by restoring the previous pin commit.

## Local customizations (expect conflicts here)

- `deployment.jsonc` — account IDs, names, hostname, Access settings. Always yours.
- `packages/custom-gatekeeper/` — organization APIs and capabilities.
- `docs/observability.md` destinations / error-reporting config.

## Repository hygiene

- Branch protection on `main` requires review; signed commits are required (`commit.gpgsign=true`).
- Never commit secrets: secrets live in Wrangler secrets, not in this repo. Credential JSON files stay untracked.
