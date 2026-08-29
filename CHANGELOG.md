# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/)
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [psake 1.1.0] - 2026-08-29

### Added

- psake skill: Build-result reference for `PsakeBuildResult`,
  `PsakeTaskResult`, diagnostics, and output-format contracts.

### Changed

- psake skill: LLM invocation guidance now prefers a quiet-capable
  `build.ps1` entry point and minimizes emitted result data.
- psake skill: Output guidance distinguishes `Quiet`, `JSON`,
  `GitHubActions`, and `Annotated` by their intended consumer.


## [2.2.0] - 2026-04-22

### Added

- psake skill: Programmatic Invocation section with `-Quiet` as the
  primary pattern for LLM agents, CI steps, and scripts — explains
  why and shows `PsakeBuildResult` usage
- psake skill: `build.ps1` entry-point template with
  `ArgumentCompleter` for live task tab completion via
  `Get-PSakeScriptTasks` (degrades gracefully if psake not loaded)
- psake skill: PSDepend try-import-first bootstrap in `build.ps1`
  template — skips install when modules are already cached, avoiding
  file-lock contention on shared CI module caches

### Changed

- psake skill: Quick Start now shows `-Quiet` programmatic pattern
  alongside the interactive form
- psake skill: Azure Pipelines CI example converted from backtick
  line continuation to splat

### Fixed

- marketplace: correct plugin source paths

## [2.1.1] - 2026-04-07

### Changed

- Both skills: description now uses third-person trigger format
  ("This skill should be used when the user asks to...")
  for more reliable skill activation
- powershellbuild skill: moved Complete Example and CI/CD
  Integration to `references/` for progressive disclosure
- powershellbuild skill: added References section pointing to
  `references/complete-example.md` and `references/ci-cd.md`

### Fixed

- powershellbuild evals: added assertions to all 3 evals,
  matching the psake eval format for automated validation

## [2.1.0] - 2026-04-01

### Added

- psake skill: declarative Task syntax with validated keys
  (`Task 'Build' @{ DependsOn = 'Clean'; Action = {...} }`)
- psake skill: file-based caching via `Inputs`/`Outputs` for
  faster incremental builds
- psake skill: structured `PsakeBuildResult` output and
  `-OutputFormat JSON|GitHubActions` for CI integration
- psake skill: `Version 5` declaration to pin build scripts
- psake skill: testability APIs (`Get-PsakeBuildPlan`,
  `Test-PsakeTask`, `-CompileOnly`)
- psake skill: hashtable `Properties` syntax
- psake skill: `exec` timeout and new process parameters
- New reference: `upgrading-to-v5.md` migration guide with
  breaking changes, caching strategies, and before/after examples

### Changed

- psake skill: SKILL.md rewritten as v5-first with v4
  backward-compatibility notes
- psake skill: `advanced.md` updated with `-OutputFormat
  GitHubActions` CI examples, removed obsolete OutputHandler section
- psake skill: `build-types.md` .NET example updated to v5
  declarative syntax with caching
- psake skill: `powershell-modules.md` bumped psake requirement
  to 5.0.0

### Removed

- psake skill: custom logging/OutputHandler documentation (removed
  in psake v5, replaced by `-OutputFormat` parameter)
