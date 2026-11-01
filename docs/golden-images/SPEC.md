# Golden Images in Backstage: specification

This is the contract every phase builds against. If you need to deviate, change this
file in the same branch and say why in `PROGRESS.md`.

Visual reference: `docs/golden-images/mockups/*.png` (layout and content only; build with
Backstage components and the app's theme, not the mockup CSS).

## 1. Scope

**In this repo (the Backstage app):**
- Catalog: an entity provider that turns each golden image Dockerfile into a `Component`,
  a status processor, and an adopter processor.
- A `golden-images` backend plugin for live data (registry, scanner, GitHub remediation status)
  and owner actions (rebuild, ask Claude to fix).
- A `golden-images` frontend plugin: Golden Images page, Remediation queue page, entity cards.
- A scaffolder module with two actions for upgrade PRs.

**Not in this repo:**
- The golden-images repo's own CI (validate-labels, build-images, remediate-cves).
- Any AI in service repos.
- Writing `catalog-info.yaml` files for golden images. There are none; the provider is the source.

"acme" is a placeholder everywhere. Every org-specific string is config or a single constant.

## 2. The golden-images repo contract (input)

```
<images repo>/
  CODEOWNERS
  images/<dir>/Dockerfile        # one folder per image
  images/<dir>/README.md         # optional
  images/<dir>/docs/             # optional, TechDocs
```

### 2.1 Labels

Read from the **final stage** of the Dockerfile. Prefix `<p>` = config `goldenImages.labelPrefix`
(default `io.acme.golden`).

| Label | Required | Values | Maps to |
|---|---|---|---|
| `org.opencontainers.image.title` | yes | text | `metadata.title` |
| `org.opencontainers.image.description` | yes | one line | `metadata.description` |
| `<p>.lifecycle` | yes | `production` \| `experimental` \| `deprecated` | `spec.lifecycle` |
| `<p>.tier` | yes | `supported` \| `best-effort` | annotation `tier` |
| `<p>.eol` | yes | `YYYY-MM-DD` | annotation `eol-date` |
| `<p>.tags` | yes | comma list, each `[a-z0-9-]+` | `metadata.tags` |
| `<p>.successor` | no | entity name, e.g. `python-3-12` | annotation `successor` |
| `<p>.image-ref` | no (see 2.3) | `registry/repo:stream`, e.g. `ghcr.io/acme/python:3.12` | annotation `image-ref` |

Values may use `ARG` defaults (`${PYTHON_VERSION}`); resolve them. Unresolvable is a validation error.

### 2.2 Derived fields (no label needed)

| Field | Rule |
|---|---|
| `metadata.name` | folder name, lowercased, every run of chars outside `[a-z0-9-]` becomes `-` (`python-3.12` → `python-3-12`) |
| `spec.owner` | `CODEOWNERS`: GitHub semantics, the **last** matching pattern for `images/<dir>/Dockerfile` wins; first owner only; mapped with `goldenImages.ownerMap`; fallback `goldenImages.defaultOwner` |
| `spec.dependsOn` | final-stage `FROM`, ARGs resolved, digest and tag stripped, matched against other images' `image-ref` repo+stream. No match = root image, no relation |
| annotation `runtime-version` | first `ARG <NAME>_VERSION=<value>` with a default, searching the final stage, then global ARGs (before the first `FROM`), then earlier stages in order; formatted `<name lowercased> <value>` (`python 3.12.7`); omitted if none |
| `spec.system` | `goldenImages.system.name` |
| `spec.type` | `golden-image` |

### 2.3 Image reference resolution

Needed to match service `FROM` lines and base images to entities, and to query the registry.
In order:
1. `<p>.image-ref` label.
2. `goldenImages.imageRefs.<dir>` in config.
3. Nothing else. Folder names don't map reliably to registry paths (`debian-12-base` is published as
   `ghcr.io/acme/base:debian-12`), so there is no naming-convention guess. Without 1 or 2 the entity
   is emitted with `metadata-incomplete` and has no lineage or adopters.

Matching ignores digest and compares `registry/repo` plus tag **stream**: a ref's tag matches a stream
if it equals it or starts with `<stream>-` (`3.12-2026.10.27` matches stream `3.12`).

### 2.4 Validation

Missing or invalid required labels never drop an image. The entity is still emitted with:
- annotation `metadata-incomplete: "true"`
- annotation `metadata-errors`: JSON array of strings, e.g. `["missing io.acme.golden.eol"]`
- a warning log line, and a counter metric if the app has metrics

## 3. Entities (output)

### 3.1 Annotation keys

All keys live in `plugins/golden-images-common/src/annotations.ts`. Namespace constant
`GOLDEN_IMAGES_ANNOTATION_PREFIX = 'golden-images.acme.io/'` (rename in one place).

On golden-image entities:

| Key (after prefix) | Set by | Value |
|---|---|---|
| `image-ref` | provider | `ghcr.io/acme/python:3.12` |
| `registry` | provider | `ghcr.io/acme/python` |
| `tier` | provider | `supported` \| `best-effort` |
| `eol-date` | provider | `2028-10-31` |
| `successor` | provider | entity name |
| `runtime-version` | provider | `python 3.12.7` |
| `dockerfile-path` | provider | `images/python-3.12/Dockerfile` |
| `metadata-incomplete`, `metadata-errors` | provider | see 2.4 |
| `cve-summary` | status processor | `C/H/M/L`, e.g. `0/1/4/11` |
| `last-scan` | status processor | ISO timestamp |
| `remediation-status` | status processor | `none` \| `queued` \| `running` \| `pr-open` \| `needs-human` \| `manual-only` \| `frozen` |
| `remediation-ref` | status processor | URL of the run or PR |

Plus standard ones: `backstage.io/managed-by-location`, `backstage.io/managed-by-origin-location`
(`url:<repo>`), `backstage.io/source-location` (`url:<repo>/tree/<branch>/images/<dir>/`),
`backstage.io/techdocs-ref` (`url:<repo>/tree/<branch>/images/<dir>`, only when `docs/` exists),
`github.com/project-slug`.

On service entities (adopter processor):

| Key | Value |
|---|---|
| `base-image` | entity ref, `component:default/python-3-12` |
| `base-image-ref` | full ref as pinned, `ghcr.io/acme/python:3.12-2026.09.15@sha256:4c88…` |
| `base-image-dockerfiles` | comma list of paths that reference it |

Never put tag lists, digests of the golden image, or CVE lists on entities. Those are fetched live.

### 3.2 Example

`docs/golden-images/fixtures/expected/python-3-12.entity.json` is the exact entity the provider must
emit for `fixtures/golden-images-repo/images/python-3.12/Dockerfile` with
`fixtures/app-config.fixtures.yaml`. Use it as a test snapshot.

### 3.3 System

The provider also emits `System` `goldenImages.system.name` owned by `goldenImages.system.owner`,
so the images repo needs no Backstage files at all.

### 3.4 Lifecycle and remediation policy

Derived, not configured per image:
- `deprecated` lifecycle → `remediation-status: frozen`
- tier `best-effort` → `manual-only` unless a run or PR exists
- tier `supported` → whatever the GitHub state says

## 4. Backend

### 4.1 `plugins/catalog-backend-module-golden-images` (pluginId `catalog`)

- **`GoldenImagesEntityProvider`**: on a scheduled task (`goldenImages.schedule`, default every
  10 min), reads the source, emits `applyMutation({ type: 'full' })`. Two sources behind one interface:
  - `url`: `UrlReader.readTree(<repo>/tree/<branch>)` filtered to `CODEOWNERS` and `images/*/Dockerfile`
    (and `images/*/docs/` existence). Uses the app's GitHub integration.
  - `dir`: a local directory, for development and tests. Links and annotations still use
    `goldenImages.source.url` + `branch`, so entities are identical whichever source is read
    (the snapshot test depends on this).
  - Optional: if the events service is installed, also refresh on a GitHub `push` event that touches
    `images/**`. Schedule alone is acceptable.
- **`GoldenImageStatusProcessor`** (Phase 3): `preProcessEntity` for `Component` with `spec.type ===
  'golden-image'`; adds the `cve-summary`, `last-scan`, `remediation-*` annotations by calling the
  golden-images backend (`/status/:name`) through `discovery` + plugin-to-plugin auth. Cached; a failed
  call leaves the entity unchanged.
- **`GoldenImageAdopterProcessor`** (Phase 5): for `Component` entities that are not golden images and
  have `backstage.io/source-location`, read Dockerfiles (in development, a repo URL listed in
  `goldenImages.adopters.devRepoDirs` is read from that local directory instead) (`Dockerfile`, `**/Dockerfile*`, max depth 3,
  or the paths in annotation `golden-images.acme.io/dockerfiles` if present), parse final-stage
  `FROM`s, match against `/images/index` from the golden-images backend, emit `dependsOn` /
  `dependencyOf` relations and the service annotations in 3.1. Use ETags; never fail processing.

Dockerfile parsing uses `dockerfile-ast` (ARG resolution, line continuations, multi-stage,
`--platform`). Put parsing in `golden-images-common` only if the frontend needs it; otherwise in a
`golden-images-node` package shared by the backend packages.

### 4.2 `plugins/golden-images-backend` (pluginId `golden-images`)

Routes (all JSON):

| Route | Purpose |
|---|---|
| `GET /images/index` | `[{ entityRef, imageRef, registry }]` from the catalog, cached |
| `GET /images/:name/registry` | recommended tag, digest, platforms, size, created, signature, SBOM URL, recent tags |
| `GET /images/:name/security` | counts by severity, findings `[{ id, pkg, severity, installed, fixedIn, status }]`, scannedAt |
| `GET /images/:name/remediation` | status, current run, open PRs labelled `auto-remediation`, recent outcomes |
| `GET /status/:name` | compact form for the status processor |
| `GET /remediation/queue` | everything the Remediation page needs |
| `POST /images/:name/remediate` | body `{ cves?: string[] }` → `workflow_dispatch` on `remediate-cves.yml` |
| `POST /images/:name/rebuild` | `workflow_dispatch` on `build-images.yml` with `image` input |

Clients behind interfaces, chosen by `goldenImages.backend.mode`:
- `RegistryClient` (OCI distribution API; ghcr first)
- `ScannerClient` (Trivy; source configurable)
- `GitHubRemediationClient` (Actions runs, PRs by label, `workflow_dispatch`) via `@backstage/integration`
- `mock` implementations backed by `docs/golden-images/fixtures/backend-mock/*.json`, used in tests and
  local dev. **Build mock first.**

Use `coreServices.cache` for registry/scanner/GitHub responses (TTL config, default 5 min).

### 4.3 Permissions

`golden-images.image.rebuild`, `golden-images.image.remediate`, resource type `catalog-entity`.
Default policy: allow when the user is an owner of the entity (`isEntityOwner`), deny otherwise.
POST routes enforce it server-side; the UI only hides buttons.

### 4.4 `plugins/scaffolder-backend-module-golden-images`

- `golden-images:resolve-tag` input `{ image: entityRef, tag?: string }` → output
  `{ registry, tag, digest, ref }` where `ref` = `registry:tag@digest`
  (recommended tag when `tag` omitted).
- `golden-images:rewrite-from` input `{ path, files?: string[], fromImage: registry, toRef }` →
  rewrites matching final-stage `FROM` lines in the workspace, output `{ changed: string[] }`.
- Example template `upgrade-base-image` lives in `docs/golden-images/fixtures/templates/` for now;
  long term it belongs in the images repo.

## 5. Frontend: `plugins/golden-images`

Detect in Phase 0 whether the app uses the legacy frontend (`createApp` from `@backstage/app-defaults`)
or the new frontend system, and follow it.

| Piece | Mockup | Notes |
|---|---|---|
| `GoldenImagesPage` at `/golden-images` | 01 | Catalog table, `type=golden-image` pinned. Columns: Name (+image-ref), Runtime, Lifecycle, Tier, CVEs (C/H/M), Last rebuild, Adopters, Remediation. Deprecated rows show successor. Buttons: Request a new image, Remediation queue |
| Entity page branch `isGoldenImage` | 02 | Tabs: Overview, Security, Remediation, Versions, Adopters, CI/CD, Dependencies, Docs |
| `GoldenImageCard` | 02 | registry, recommended tag, digest, platforms, size, copy FROM / pull |
| `ImageSecurityCard` | 02 | severity counts, top findings with remediation state, signature, SBOM, "Ask Claude to fix" (permission-gated) |
| `ImageLifecycleCard` | 02 | lifecycle, tier, EOL, rebuild cadence, auto-remediation policy, Rebuild (permission-gated) |
| `AdoptersCard` | 02 | services via `dependencyOf`, on latest vs behind, link to bulk upgrade |
| `BaseImageCard` (services) | 03 | from `base-image*` annotations; behind-by and fixed CVEs; "fix in review" note from `/remediation` |
| `RemediationQueuePage` at `/golden-images/remediation` | 07 | columns Queued / Claude working / PR in review / Shipped / Needs human; policy panel; recent outcomes |

A `goldenImagesApiRef` client wraps the backend routes using `discoveryApi` + `fetchApi`.
Use core components (`InfoCard`, `Table`, `StatusOK`/`StatusWarning`/`StatusError`, `Link`,
`CopyTextButton`) and the theme. Empty, loading and error states for every card.

## 6. Config (`config.d.ts`)

```yaml
goldenImages:
  source:
    url: https://github.com/acme/golden-images   # canonical repo; always used for links
    branch: main
    # dir: ./docs/golden-images/fixtures/golden-images-repo   # if set, read files from here instead
  labelPrefix: io.acme.golden
  registryPrefix: ghcr.io/acme/
  imageRefs: {}                 # dir -> image ref overrides
  system:
    name: golden-images
    owner: group:default/platform-team
  ownerMap:
    '@acme/platform-team': group:default/platform-team
  defaultOwner: group:default/platform-team
  schedule:
    frequency: { minutes: 10 }
    timeout: { minutes: 3 }
  adopters:
    devRepoDirs: {}             # dev only: repo URL -> local dir
  backend:
    mode: mock                  # mock | live
    cacheTtl: { minutes: 5 }
    github:
      repo: acme/golden-images
      remediateWorkflow: remediate-cves.yml
      rebuildWorkflow: build-images.yml
      remediationLabel: auto-remediation
    registry: {}                # live mode details, Phase 7
    scanner: {}                 # live mode details, Phase 7
```

## 7. Package names

`@internal/` scope unless Phase 0 finds the app uses another.
- `plugins/golden-images-common` (types, annotation constants, ref matching): isomorphic
- `plugins/golden-images-node` (Dockerfile parsing, CODEOWNERS): backend only, if needed
- `plugins/catalog-backend-module-golden-images`
- `plugins/golden-images-backend`
- `plugins/scaffolder-backend-module-golden-images`
- `plugins/golden-images`
