# Berty Windows GUI Port — Project Log

> Status: exploratory / audit phase  
> Repository: `bellingcatosint/berty`  
> Working branch: `docs/windows-gui-porting-log`  
> Baseline: `master` at `de4ae4826b4b60a95b273b6c6ca997273618315a`  
> Started: 2026-11-08

## 1. Purpose

Investigate and, if technically viable, produce a usable Windows desktop build of Berty by reusing the GUI code already present in the repository. Prefer the smallest maintainable change over a new client implementation or a rewrite of the application architecture.

This is an independent effort in the `bellingcatosint/berty` repository. It is not an official Berty upstream commitment or a claim of upstream support.

## 2. Working channels

- Project communication chat: https://t.me/expanse_org
- The chat link was supplied by the project owner on 2026-11-08. The link could not be independently previewed during this update; its designation as the working chat is based on the owner's statement.

## 3. Working rules

1. Keep the default branch untouched during development. Work on a dedicated branch and propose changes through a pull request.
2. Record each meaningful inspection, change, test, failure, and deviation in this log.
3. Separate verified repository facts from assumptions and proposals.
4. Never report a build as working until the executable has been run and the relevant functionality tested.
5. Distinguish compilation, GUI startup, local-node operation, peer connectivity, and message exchange as separate test results.
6. If the plan changes, record what changed, why, and what evidence caused the change.
7. Do not treat Berty as suitable for sensitive communications merely because a Windows build launches. Security suitability requires a separate, current assessment.

## 4. Initial repository findings

The following are initial code/configuration observations. They are not yet proof that a Windows GUI build works.

- The repository contains an experimental Fyne-based GUI entry point in `go/cmd/berty/gui.go`.
- That file is guarded by the Go build tag `bertygui`.
- The GUI is exposed through the `berty gui` command and is described in the source as experimental and not officially supported.
- The Windows GoReleaser configuration currently defines a Windows amd64 CLI build; it does not currently define the GUI build.
- The Linux GoReleaser configuration contains a separate GUI build using the `bertygui` build tag. This is evidence of an existing packaging pattern, not proof that the same configuration will work on Windows.
- The release workflow includes a `windows-latest` runner in its build matrix. The exact toolchain and CGO prerequisites still need to be checked.
- The mobile Expo/React Native project is not, by itself, a Windows desktop package.

## 5. Objective and proposed delivery sequence

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

## 6. Current status

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

## 7. Decision log

### 2026-11-08 — Initial direction

**Decision:** investigate the existing Fyne GUI before considering Electron, Tauri, a React Native desktop port, or a rewrite.

**Reason:** a GUI entry point already exists in the Go application. Reusing it may be the shortest path, but Windows-specific CGO/Fyne requirements and runtime behavior remain unverified.

**Confidence:** medium for the existence of a reusable GUI path; low for the effort and reliability of a Windows release until the audit and build tests are complete.

## 8. Next actions

1. Inspect the complete Windows release workflow, including compiler installation and CGO environment.
2. Compare Windows and Linux GoReleaser configurations, including archive IDs and naming.
3. Inspect Fyne dependencies and any platform-specific code reached by the GUI build.
4. Run or reproduce the proposed Windows build in CI.
5. Update this log with exact commands, commit references, test evidence, and blockers.

## 9. Audit notes — 2026-11-08

### Verified from the checked-in configuration

- The release matrix contains a `windows-latest` job, but the workflow also contains the comment `TODO: bertyguy`. The presence of a Windows runner therefore does not mean the GUI is already built there.
- The Windows job installs Go with `actions/setup-go`, but the inspected Windows-specific steps do not install or configure a C compiler.
- The Linux dependency step installs MinGW and Linux graphics/X11 development packages, but that step is conditional on `goos == linux` and does not run on the Windows job.
- The Windows GoReleaser file currently has one generic archive and one build with ID `windows`.
- The Linux GoReleaser file uses separate build IDs and separate archives for CLI (`berty`) and GUI (`bertygui`). A Windows GUI addition should follow the explicit artifact-separation pattern rather than simply appending a second build to the current generic archive.
- The GUI source imports Fyne's app package and runs a native Fyne application. This makes the native Windows toolchain and CGO setup a first-order audit item.

### Consequence / correction to the initial shortcut

The earlier suggestion that the Windows GUI could be enabled by adding only a GoReleaser build stanza was incomplete. The workflow currently has an explicit GUI TODO and no visible Windows C-compiler setup. The shortest viable path may still be the existing Fyne GUI, but it requires a CI/toolchain audit before a safe patch can be specified.

### Not yet verified

- Whether the current pinned Go version and Fyne version build successfully on the Windows runner.
- Which Windows C compiler/toolchain is compatible with this repository's Fyne dependencies and current build setup.
- Whether the GUI can start and operate correctly on supported Windows versions.
- Whether all runtime dependencies are present on a clean Windows machine.

## 10. Change log

- 2026-11-08: Created the project log and dedicated documentation branch. No application source or release configuration changed.
- 2026-11-08: Inspected the release workflow, Windows and Linux GoReleaser configurations, and GUI entry point. Recorded a material caveat: Windows CI currently marks GUI work as TODO and does not visibly provision a C compiler in the Windows job. No application source or release configuration changed.
- 2026-11-08: Added the project owner's designated working chat, https://t.me/expanse_org. The link preview could not be independently fetched; no claim is made about its membership or access settings.

## 11. Audit notes — 2026-11-09

### Scope of this pass

Read-only inspection of the checked-in Go module metadata, Windows and Linux GoReleaser configurations, release workflow, GUI entry point, and GUI package structure. No application source, release configuration, dependency version, or default-branch file was changed. No build command was executed in this pass.

### Verified repository facts

- The root `go.mod` declares Go `1.26.4`.
- The same module directly requires `fyne.io/fyne/v2 v2.5.2`.
- The module dependency list includes the native OpenGL/GLFW Go bindings `github.com/go-gl/gl` and `github.com/go-gl/glfw/v3.3/glfw`. Their presence, together with the Fyne application imports and `CGO_ENABLED=1` in the Windows GoReleaser build, makes the C toolchain and native graphics linkage a concrete build prerequisite to investigate. This is not yet proof of the exact compiler package or version required.
- `go/cmd/berty/gui.go` is guarded by the `bertygui` build tag. It creates a Fyne application and exposes the experimental `berty gui` command.
- `go/cmd/berty/gui/main.go` creates a window, initializes the messenger store and GUI context, displays the root widget, and waits for the window to close.
- `go/cmd/berty/gui/app.go` assembles existing account, contact-request, contact-addition, multi-member, and conversation widgets. This confirms that the GUI code is more than an empty entry point; it does not verify that these functions work on Windows.
- The Windows GoReleaser configuration currently defines one build ID, `windows`, with binary name `berty`, target `windows/amd64`, and `CGO_ENABLED=1`. It has no `bertygui` build or separate GUI archive.
- The Linux GoReleaser configuration defines separate `berty` and `bertygui` builds and archives. The GUI build adds `-tags=bertygui`.
- The release workflow's Windows matrix job installs Go, then invokes GoReleaser with the Windows configuration. The workflow does not contain an explicit Windows step that installs or configures a C compiler. The Linux-only dependency step installs MinGW and Linux graphics development packages; its condition excludes the Windows runner.
- The release workflow is triggered for pull requests only when specified paths change. The current pull request changes only `docs/windows-porting/PROJECT_LOG.md`, which is outside its listed path filters. Therefore, the absence of a release workflow run for this documentation-only pull request is expected and must not be described as a failed build.

### Interpretation — kept separate from verified facts

- The existing Fyne GUI is a plausible reuse path, but the Windows build cannot be considered ready until the native toolchain and graphics linkage are demonstrated on the target runner.
- The absence of an explicit compiler-installation step is a reproducibility gap in the checked-in workflow. It does not, by itself, prove that the hosted Windows image lacks a usable compiler.
- The Linux packaging pattern is a useful reference for artifact separation, not proof that its configuration can be copied unchanged to Windows.
- No evidence has yet been collected for Windows compilation, GUI startup, local-node lifecycle, peer connectivity, or message exchange.

### Decision

Do not modify the Windows release configuration yet. First identify and verify a compatible Windows C toolchain on the actual CI runner, then run a minimal tagged GUI build. Keep the existing CLI artifact unchanged. Only after the build succeeds should a separate Windows GUI build and archive be added.

### Next verification actions

1. Inspect the exact Fyne 2.5.2 Windows build requirements and the GLFW/OpenGL dependencies selected by the current module graph.
2. Determine which C compiler is available on the hosted Windows runner by default, rather than assuming one is present or absent.
3. Check whether the existing Windows CLI build depends on CGO in this repository and whether it currently succeeds on a relevant workflow run.
4. Prepare a minimal, isolated CI experiment for `go build -tags=bertygui` targeting Windows amd64. Do not publish or release its output.
5. Record the exact runner image, Go version, compiler identity/version, command, exit code, and complete relevant error output.
6. If compilation succeeds, proceed to a separate GUI artifact definition; if it fails, record the first reproducible blocker before changing dependencies or architecture.

### Change log

- 2026-11-09: Completed a read-only audit of Go/Fyne versions, GUI source structure, native graphics bindings, and Windows/Linux packaging differences. Clarified why the documentation-only pull request did not trigger the release workflow. No application or release configuration changed; no Windows build was run.
