---
name: golden-images
description: Run one phase of the Golden Images in Backstage build plan (docs/golden-images/PLAN.md). Use as /golden-images <phase number>.
disable-model-invocation: true
---

# Run golden images phase $ARGUMENTS

## 1. Load context
Read, in this order:
- `docs/golden-images/PLAN.md`, the section for Phase $ARGUMENTS only
- `docs/golden-images/SPEC.md`
- `docs/golden-images/PROGRESS.md`
- `docs/golden-images/RECON.md` (if Phase $ARGUMENTS is not 0)
- any mockup PNG the phase mentions, from `docs/golden-images/mockups/`

Stop and tell me if:
- no phase number was given, or the phase doesn't exist;
- the phase isn't 0 and `RECON.md` is missing (run Phase 0 first);
- `PROGRESS.md` shows an earlier phase unfinished;
- RECON.md says this repo is the upstream Backstage monorepo.

## 2. Prepare the branch
- `git status` must be clean. If not, stop and ask.
- Create `golden-images/phase-$ARGUMENTS-<short-slug>` from the current default branch.

## 3. Plan, then wait
Write a short plan: packages and files to create or change, key types, test list mapped to the phase's
acceptance criteria, anything in SPEC.md that doesn't fit. Show it and **wait for my go-ahead** before
writing code. For Phase 0 the plan is just the list of things you'll inspect.

## 4. Build
- Follow `docs/golden-images/CONTEXT.md`.
- Check Backstage APIs in `node_modules` types before using them.
- Use `yarn new` for new packages where the generator exists.
- Write the tests the phase lists, using `docs/golden-images/fixtures/`.

## 5. Verify
Run the phase's acceptance commands (exact commands are in RECON.md). Fix until they pass. Don't weaken
tests. If a manual check is listed, tell me how to do it; don't claim it passed.

## 6. Record and commit
- Append the phase entry to `docs/golden-images/PROGRESS.md` (template is in the file).
- If SPEC.md changed, say so at the top of the entry.
- Commit with a message `feat(golden-images): phase $ARGUMENTS – <summary>`. Don't push.

## 7. Report
Reply with: what was built, check results, deviations from SPEC, the manual check steps, and what the
next phase needs from me (config, credentials, decisions).
