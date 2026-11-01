# Golden images feature: working rules

Imported from AGENTS.md (which CLAUDE.md imports). Applies to all work on the golden images feature.

## Where things are
- Contract: `docs/golden-images/SPEC.md`. Plan: `docs/golden-images/PLAN.md`.
- Repo facts (versions, commands, frontend/backend system): `docs/golden-images/RECON.md`.
- What's done and what deviated: `docs/golden-images/PROGRESS.md`. Read it before starting a phase.
- Visual reference: `docs/golden-images/mockups/*.png`. Look at the one a phase names before building UI.
- Test fixtures: `docs/golden-images/fixtures/`. Tests use these; never the network.

## Rules
- Work one phase at a time via `/golden-images <n>`. Don't start work from a later phase.
- Check Backstage APIs against the installed `@backstage/*` types in `node_modules`, not memory.
  Versions move; import paths (e.g. `/alpha`) change between releases.
- Prefer `yarn new` generators for new packages, then edit.
- Annotation keys come only from `golden-images-common`'s constants. No string literals elsewhere.
- No `catalog-info.yaml` for golden images. The provider is the only source.
- Tags, digests and CVE lists are fetched live; only the summary annotations in SPEC 3.1 go on entities.
- "acme" is a placeholder: anything org-specific is config or one constant.
- Build mock clients before live ones. Tests never make real network calls.
- UI: use Backstage core components and the app theme. The mockups are for layout and content, not CSS.
- Every card has loading, error and empty states.
- Owner-only actions are enforced server-side with the permission framework; hiding a button isn't enough.
- Ask before adding a dependency not named in SPEC.md.
- Don't edit production config. Local settings go in `app-config.local.yaml`.
- Don't weaken or delete a failing test to get green. Fix the code or explain why the test is wrong.
- If the spec doesn't fit what you find, update SPEC.md in the same branch and record why in PROGRESS.md.
