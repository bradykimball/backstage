# Golden images: Phase 0 recon

Date: 2026-11-01 · branch `golden-images/phase-0-recon` · repo `bradykimball/backstage`

## Verdict: STOP. This is the upstream Backstage monorepo

This repo is a fork of `backstage/backstage`, not a Backstage app. Evidence:

- Root `package.json` is named `root`, version `1.45.0-next.0`, and `repository.url` is
  `https://github.com/backstage/backstage`.
- `backstage.cli.new.globals.namePrefix` is `@backstage/`, so `yarn new` would generate packages
  under the published `@backstage/*` scope.
- 153 `plugins/*` and about 50 `packages/*`, published as `@backstage/*` and wired with `workspace:^`.
- `.changeset/` is present (26 entries) along with API reports (`build:api-reports`). There is no
  `backstage.json`: apps have one, the monorepo does not.

Internal plugins like golden images belong in a company app created with
`npx @backstage/create-app@latest`, which depends on released `@backstage/*` versions. Building
them here would put internal code in the upstream source tree, make every upstream rebase a
conflict risk, and give the packages the `@backstage/*` scope.

**Recommendation:** create an app (`npx @backstage/create-app@latest`) and install the kit there.
Either unzip it again or copy `docs/golden-images/`, `.claude/`, `AGENTS.md` and `CLAUDE.md` from
the `golden-images/kit` branch. Then re-run `/golden-images 0`. Don't start Phase 1 in this repo.

## Facts recorded anyway (in case you proceed here regardless)

| Item | Finding |
|---|---|
| `@backstage/*` versions | No `backstage.json`. Everything is `workspace:^` at `1.45.0-next.0` (pre-release). |
| Backend system | New system: `createBackend()` from `@backstage/backend-defaults` (`packages/backend/src/index.ts`). |
| Frontend system | `packages/app`: legacy `createApp` from `@backstage/app-defaults`. `packages/app-next`: new frontend system. |
| Catalog rules | `catalog.rules.allow`: Component, API, Resource, System, Domain, Location. |
| Catalog providers | `azureBlob` and `backstageOpenapi` in `app-config.yaml`. Locations are only examples. |
| GitHub integration | Yes: `integrations.github` for `github.com` with `${GITHUB_TOKEN}`, plus `scaffolder-backend-module-github` and a GitHub auth module. |
| Events backend | Yes: `@backstage/plugin-events-backend` and the google-pubsub module. |
| Permissions | `permission.enabled: true`. Policy: `plugin-permission-backend-module-allow-all-policy` (allow-all). |
| Typecheck | `yarn tsc` (full check: `yarn tsc:full`) |
| Test | `yarn test` (repo-wide, `backstage-cli repo test`). Per package: `yarn workspace <name> test` |
| Lint | `yarn lint` (changes since `origin/master`). Whole repo: `yarn lint:all` |
| Start | `yarn start` (`backstage-cli repo start`) |
| `yarn new` templates | `catalog-provider-module`, `backend-plugin`, `backend-plugin-module`, `frontend-plugin`, `new-frontend-plugin`, `new-frontend-plugin-module`, `scaffolder-backend-module`, `plugin-common-library`, `plugin-node-library`, `plugin-web-library`, `node-library`, `web-library`. The kit's `common-library` corresponds to `plugin-common-library`. |
| Package scope | `@backstage/` (from `namePrefix`). There are no `@internal/` packages. |
| `node_modules` | Not installed in this checkout. Run `yarn install` before any build. |

## SPEC.md fit

- SPEC 6 assumes the `@internal/` scope, but this repo's generator defaults to `@backstage/`. An
  app made with `create-app` uses `@internal/` by default, so the SPEC fits as written there.
- The plugin paths in SPEC 4–5 (`plugins/golden-images*`) also fit an app layout.
- No SPEC changes proposed until recon runs in the real app.
