# Berty Windows GUI Port — Project Log

> Status: exploratory / audit phase  
> Repository: `bellingcatosint/berty`  
> Working branch: `docs/windows-gui-porting-log`  
> Baseline: `master` at `de4ae4826b4b60a95b273b6c6ca997273618315a`  
> Started: 2026-11-08

## 1. Purpose

Investigate and, if technically viable, produce a usable Windows desktop build of Berty by reusing the GUI code already present in the repository. Prefer the smallest maintainable change over a new client implementation or a rewrite of the application architecture.

This is an independent effort in the `bellingcatosint/berty` repository. It is not an official Berty upstream commitment or a claim of upstream support.

## 2. Working rules

1. Keep the default branch untouched during development. Work on a dedicated branch and propose changes through a pull request.
2. Record each meaningful inspection, change, test, failure, and deviation in this log.
3. Separate verified repository facts from assumptions and proposals.
4. Never report a build as working until the executable has been run and the relevant functionality tested.
5. Distinguish compilation, GUI startup, local-node operation, peer connectivity, and message exchange as separate test results.
6. If the plan changes, record what changed, why, and what evidence caused the change.
7. Do not treat Berty as suitable for sensitive communications merely because a Windows build launches. Security suitability requires a separate, current assessment.

## 3. Initial repository findings

The following are initial code/configuration observations. They are not yet proof that a Windows GUI build works.

- The repository contains an experimental Fyne-based GUI entry point in `go/cmd/berty/gui.go`.
- That file is guarded by the Go build tag `bertygui`.
- The GUI is exposed through the `berty gui` command and is described in the source as experimental and not officially supported.
- The Windows GoReleaser configuration currently defines a Windows amd64 CLI build; it does not currently define the GUI build.
- The Linux GoReleaser configuration contains a separate GUI build using the `bertygui` build tag. This is evidence of an existing packaging pattern, not proof that the same configuration will work on Windows.
- The release workflow includes a `windows-latest` runner in its build matrix. The exact toolchain and CGO prerequisites still need to be checked.
- The mobile Expo/React Native project is not, by itself, a Windows desktop package.

## 4. Objective and proposed delivery sequence

### Phase 0 — Audit (current)

- Inspect the Windows release workflow and all relevant GoReleaser configuration.
- Trace Fyne and CGO requirements for Windows amd64.
- Check whether the GUI code and its dependencies compile under the repository's current Go/toolchain versions.
- Identify packaging, runtime, and platform-specific blockers.

**Exit condition:** a documented, evidence-backed build plan, or a clear explanation of why the existing GUI cannot be used.

### Phase 1 — Minimal Windows GUI build

If the audit supports it:

- Add a Windows GUI build using `-tags=bertygui`.
- Keep the existing CLI artifact intact.
- Ensure GUI and CLI artifacts have unambiguous names and packaging rules.
- Use the existing Windows CI runner where practical.

**Exit condition:** CI produces a distinct Windows amd64 GUI artifact without regressing the CLI build.

### Phase 2 — Runtime validation

Test and record separately:

- executable starts on a clean supported Windows environment;
- GUI window initializes and remains responsive;
- local node lifecycle works;
- connection to another node works;
- message exchange works;
- shutdown and restart behave correctly;
- logs and failure states are usable for diagnosis.

**Exit condition:** reproducible test results, including environment details and known limitations.

### Phase 3 — Distribution

- Publish a versioned ZIP artifact first.
- Consider an installer (for example MSI) only after the ZIP build and runtime tests are reliable.
- Document installation, first launch, connection setup, troubleshooting, and removal.

## 5. Current status

| Area | Status | Notes |
|---|---|---|
| Existing GUI source | Found | Fyne GUI behind `bertygui` build tag |
| Windows CLI packaging | Found | Existing Windows amd64 CLI configuration |
| Windows GUI packaging | Missing in inspected config | To be evaluated and implemented if viable |
| Windows CI runner | Found | CGO/compiler/toolchain details not yet validated |
| Windows compilation | Not tested | No successful build claim |
| Windows runtime | Not tested | No successful launch claim |
| Peer/message functionality | Not tested | No functionality claim |
| Installer | Not started | Deferred until ZIP/runtime validation |

## 6. Decision log

### 2026-11-08 — Initial direction

**Decision:** investigate the existing Fyne GUI before considering Electron, Tauri, a React Native desktop port, or a rewrite.

**Reason:** a GUI entry point already exists in the Go application. Reusing it may be the shortest path, but Windows-specific CGO/Fyne requirements and runtime behavior remain unverified.

**Confidence:** medium for the existence of a reusable GUI path; low for the effort and reliability of a Windows release until the audit and build tests are complete.

## 7. Next actions

1. Inspect the complete Windows release workflow, including compiler installation and CGO environment.
2. Compare Windows and Linux GoReleaser configurations, including archive IDs and naming.
3. Inspect Fyne dependencies and any platform-specific code reached by the GUI build.
4. Run or reproduce the proposed Windows build in CI.
5. Update this log with exact commands, commit references, test evidence, and blockers.

## 8. Change log

- 2026-11-08: Created the project log and dedicated documentation branch. No application source or release configuration changed.
