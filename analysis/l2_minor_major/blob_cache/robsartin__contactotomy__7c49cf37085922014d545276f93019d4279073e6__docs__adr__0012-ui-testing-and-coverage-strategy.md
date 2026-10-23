# 12. UI testing and coverage strategy

Date: 2026-06-24

## Status

Accepted. Supersedes the coverage-threshold figures in ADR-0009 (raises them).

## Context

Plan 4 introduces a Compose Desktop UI. The repo enforces code-coverage floors
(ADR-0009 originally set line ≥80%, branch ≥60%). We want the UI to be genuinely
tested — not excluded from coverage — and we want to hold a higher bar now that the
codebase is well-established. Measured coverage at this point is line 93.7% /
branch 71.9%, so there is headroom to raise the floors.

Compose Multiplatform provides an offscreen JUnit test harness
(`compose.desktop.uiTestJUnit4`, `runComposeUiTest { setContent { … } }`) that
renders composables without a real window and runs headlessly on CI. This lets us
drive screens (clicks, text assertions, state changes) and cover the rendering
code, rather than excluding it.

## Decision

- **Raise the coverage floors to line ≥ 90%, branch ≥ 70%** (updates ADR-0009's
  figures). Still floors to raise over time, never lowered to make a build pass.
- **Logic lives in framework-free state holders** (e.g. `AppStore`) and pure
  helpers, separate from composables; unit-tested as before.
- **Composables are tested with Compose UI tests** (`runComposeUiTest`) and count
  toward coverage. Each UI sub-plan ships UI tests that exercise its screens.
- **Only genuinely untestable seams are excluded from coverage:** the application
  entry point (`MainKt` / `application {}`) and thin platform adapters such as the
  real AWT `FileDialog` implementation (isolated behind the `FilePicker`
  interface). The exclusion list stays scoped to these; it is never used to hide
  untested logic.
- **Run-the-app verification remains** part of each UI sub-plan's "done"
  (`./gradlew run` + a screenshot/notes in the PR) as a complement to UI tests.

## Consequences

- Higher floors (90/70) keep quality rising; current coverage clears them. (The
  branch floor was raised 65 → 70 on 2026-06-26 once coverage was comfortably above
  it; the line floor stays 90.)
- The UI is actually tested, headlessly and in CI, not assumed-correct.
- Composables must stay thin and driven by injectable state so `runComposeUiTest`
  can exercise them with fakes (no real window/file dialog).
- CI must be able to run the Compose offscreen test harness headlessly; if Skiko
  needs a virtual display on the Linux runner, the CI workflow adds it (e.g.
  `xvfb-run`), decided during implementation.
- ADR-0009 remains in force except for the threshold figures, which this ADR
  raises.
