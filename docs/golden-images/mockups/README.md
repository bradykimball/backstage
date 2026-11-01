# Mockups

Layout and content reference for the UI phases. Build with Backstage core components and the
app's theme; don't copy the mockup CSS.

| File | Used in phase | Notes |
|---|---|---|
| 01-golden-images-page.png | 2, 3, 5 | Dedicated page, custom columns |
| 02-image-entity-page.png | 2, 4, 7 | Golden image entity page, six cards |
| 03-service-base-image-card.png | 5 | Base image card on a service page |
| 04-upgrade-base-image-template.png | 6 | Scaffolder form and PR preview |
| 05-golden-images-repo.png | reference | The images repo (not built here) |
| 06-build-pipeline.png | reference | Images repo CI (not built here) |
| 07-remediation-queue.png | 7 | Remediation queue page |
| 08-claude-remediation-run.png | reference | Images repo remediation workflow (not built here) |
| 09-claude-remediation-pr.png | reference | What reviewers see in the images repo |

Sample data in the mockups (node-20, go-1.23, java-21-temurin) differs slightly from the test
fixtures (node-22, go-1.26). When they disagree, the fixtures and SPEC.md win.
