# Golden Images in Backstage: build plan

Run one phase per Claude Code session and per branch: `/golden-images <phase>`.
Each phase leaves the app working and mergeable. Later phases only add.

Acceptance commands below say `<typecheck>`, `<test pkg>`, `<lint pkg>`: use the exact commands
recorded in `RECON.md` (Phase 0). In a standard app they are `yarn tsc`,
`yarn backstage-cli package test` / `yarn test` inside the package, and `yarn lint`.

---

## Phase 0 · Recon (no code)

**Goal:** know the repo before changing it.

Write `docs/golden-images/RECON.md` with:
- What this repo is. **If it is the upstream Backstage monorepo** (root `package.json` named `root`,
  dozens of `plugins/*` published as `@backstage/*`, `.changeset/`, API reports), stop and say so:
  internal plugins belong in a company app created with `npx @backstage/create-app@latest`, not in a
  fork of Backstage itself. Recommend that and end the phase.
- `@backstage/*` versions (from `backstage.json` and `packages/backend/package.json`).
- Backend system (new `createBackend` vs legacy) and frontend system (legacy `createApp` vs new).
- How the catalog is configured today: providers, locations, rules allowing `Component`/`System`.
- GitHub integration present? Events backend present? Permission framework enabled, and which policy?
- Exact typecheck / test / lint / start commands, and whether `yarn new` offers
  `catalog-provider-module`, `backend-plugin`, `frontend-plugin`, `scaffolder-backend-module`,
  `common-library`, `node-library`.
- Package scope used for internal packages.
- Anything in SPEC.md that doesn't fit this repo, with a proposed change.

**Acceptance:** RECON.md exists and answers every bullet. No other files changed.

---

## Phase 1 · Entity provider

**Goal:** golden images appear in the catalog from Dockerfiles alone.

Build:
- `golden-images-common`: annotation constants (SPEC 3.1), `GoldenImageMetadata` type,
  image-ref parsing and stream matching (SPEC 2.3), `isGoldenImage(entity)`.
- Dockerfile + CODEOWNERS parsing (`golden-images-node`, or inside the module if nothing else needs it
  yet): final stage, ARG resolution, labels, FROM, `*_VERSION` ARG.
- `catalog-backend-module-golden-images`: `GoldenImagesEntityProvider` with `dir` and `url` sources,
  System entity, validation per SPEC 2.4, schedule from config, `config.d.ts`.
- Register the module in `packages/backend`. Add an `app-config.local.yaml` example (not committed if
  the repo gitignores it) pointing `goldenImages.source.dir` at the fixtures.

Tests (fixtures in `docs/golden-images/fixtures/golden-images-repo`):
- python-3.12 entity equals `fixtures/expected/python-3-12.entity.json`.
- debian-12-base has no `dependsOn` (external base); python-3.12 and node-22 depend on it.
- go-1.26 uses the **final** stage (distroless): no `dependsOn`, tier `supported` (the builder stage's `best-effort` label is ignored), `runtime-version` `go 1.26.2` from the builder ARG.
- nginx-1.26-fips resolves `FROM ${BASE_IMAGE}` from its ARG default.
- python-3.9 is `deprecated` with `successor: python-3-12`.
- broken-missing-labels is emitted with `metadata-incomplete` and errors naming the missing labels.
- CODEOWNERS: last matching pattern wins; unknown team falls back to `defaultOwner`.
- The System entity is emitted.

**Acceptance:** `<typecheck>`, `<test pkg>` for each new package, `<lint pkg>` pass. Starting the app
locally with the fixture config shows 7 golden images + the System in the catalog, and python-3-12's
dependency graph shows debian-12-base.

**Out of scope:** any UI, any network calls except through `UrlReader` in the `url` source.

---

## Phase 2 · Golden Images page and entity page branch

**Goal:** engineers can browse golden images and open a sensible entity page.

Build (`plugins/golden-images` frontend):
- `GoldenImagesPage` at `/golden-images` (mockup 01): pinned `type=golden-image` filter; columns that
  can be filled now: Name + image-ref, Runtime, Lifecycle, Tier, EOL; deprecated row shows successor.
  Leave CVEs / Last rebuild / Adopters / Remediation columns rendering `—` from absent annotations.
- Sidebar item "Golden Images".
- Entity page branch for `isGoldenImage`: Overview with stock About, Links, catalog graph
  (dependsOn); Docs tab with TechDocs; placeholders for the tabs later phases fill.
- An "incomplete metadata" warning banner on the entity page when `metadata-incomplete` is set.

**Acceptance:** checks pass; component tests for the page (renders rows from a mocked catalog API,
deprecated row shows successor) and for the banner. Manual: page matches mockup 01's structure.

---

## Phase 3 · Backend plugin (mock mode) and status processor

**Goal:** live data paths exist end to end, backed by mocks.

Build:
- `golden-images-backend` with every GET route in SPEC 4.2, `RegistryClient` / `ScannerClient` /
  `GitHubRemediationClient` interfaces, **mock** implementations reading
  `docs/golden-images/fixtures/backend-mock/*.json`, cache, `config.d.ts`.
- `GoldenImageStatusProcessor` in the catalog module, calling `/status/:name` with plugin-to-plugin
  auth, adding `cve-summary`, `last-scan`, `remediation-status`, `remediation-ref`; lifecycle/tier rules
  from SPEC 3.4.
- `goldenImagesApiRef` client in the frontend plugin.
- Golden Images page: CVEs, Last rebuild and Remediation columns now filled.

**Acceptance:** checks pass; route tests with supertest against mock clients; processor test with a
mocked discovery/fetch (success, failure leaves entity unchanged, deprecated → frozen). Manual: the
table shows CVE counts and remediation states from the mock data.

**Out of scope:** POST routes, live clients.

---

## Phase 4 · Golden image entity cards

**Goal:** the image page from mockup 02.

Build: `GoldenImageCard`, `ImageSecurityCard`, `ImageLifecycleCard`, Remediation tab content
(current run / open PRs / recent outcomes from `/images/:name/remediation`), Versions tab
(recent tags), Security tab (full findings table). "Ask Claude to fix" and "Rebuild" buttons render but
are disabled with a tooltip until Phase 7.

**Acceptance:** checks pass; component tests for each card covering loading, error, empty and data
states. Manual: compare against mockup 02.

---

## Phase 5 · Adopters

**Goal:** know which services use which image, without anyone editing YAML.

Build:
- `/images/index` route if not already complete.
- `GoldenImageAdopterProcessor` per SPEC 4.1, tested against
  `docs/golden-images/fixtures/service-repos/*` through a fake `UrlReader`.
- `AdoptersCard` + Adopters tab (on latest vs behind, computed from `base-image-ref` vs
  `/images/:name/registry` recommended tag).
- `BaseImageCard` on service entity pages (mockup 03), shown only when `base-image` is set, including the
  "fix in review" note from `/images/:name/remediation`.
- Golden Images page: Adopters column filled.

Fixture expectations:
- payments-api → python-3-12, two Dockerfiles, pinned `3.12-2026.09.15`.
- ledger-worker → python-3-12 (final stage), not via its builder stage.
- web-frontend (`FROM node:20` from Docker Hub) → no relation.
- legacy-report → python-3-9 (deprecated).

**Acceptance:** checks pass; processor tests for all four fixtures, ETag reuse, and read failures
not failing processing. Manual: payments-api shows the Base image card; python-3-12 lists adopters.

---

## Phase 6 · Scaffolder upgrade flow

**Goal:** one click from "you're behind" to an upgrade PR.

Build: `scaffolder-backend-module-golden-images` with `golden-images:resolve-tag` and
`golden-images:rewrite-from` (SPEC 4.4) plus tests; example `upgrade-base-image` template in
`docs/golden-images/fixtures/templates/` registered as a catalog location in local config; the
"Open upgrade PR" (BaseImageCard) and "Upgrade a service" / "Open upgrade PRs for N behind"
(image page) buttons link to the template with parameters prefilled.

**Acceptance:** checks pass; action tests on a temp workspace (multi-stage, ARG FROM, two files,
no match = no change). Manual: the template's dry-run shows the expected diff for payments-api.

---

## Phase 7 · Owner actions, remediation queue, live clients

**Goal:** the Claude remediation loop is visible and triggerable from Backstage.

Build:
- Permissions per SPEC 4.3, enforced in `POST /images/:name/remediate` and `/rebuild`
  (`workflow_dispatch` through the GitHub integration).
- Enable "Ask Claude to fix" and "Rebuild" for owners.
- `RemediationQueuePage` at `/golden-images/remediation` (mockup 07) from `/remediation/queue`.
- Live `RegistryClient`, `ScannerClient`, `GitHubRemediationClient` behind `mode: live`, each with
  config documented in `config.d.ts`. Keep mock as default for local dev.

**Acceptance:** checks pass; permission tests (owner allowed, non-owner 403); dispatch tests with a
mocked Octokit; queue page component test. Manual: with `mode: mock`, the queue matches mockup 07;
with `mode: live` against a test repo, "Ask Claude to fix" starts a `remediate-cves.yml` run.
