# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/)
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

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
