# psake Skill Changelog

All notable changes to the psake skill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/)
and this skill adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.1.0] - 2026-08-29

### Added

- Build-result reference for `PsakeBuildResult`, `PsakeTaskResult`,
  diagnostics, and output-format contracts.

### Changed

- LLM invocation guidance now prefers a quiet-capable `build.ps1` entry point
  and minimizes emitted result data.
- Output guidance distinguishes `Quiet`, `JSON`, `GitHubActions`, and
  `Annotated` by their intended consumer.

## [1.0.0] - 2026-04-22

### Added

- Programmatic Invocation guidance with `-Quiet`, `PsakeBuildResult`,
  task completion, and error handling patterns.
- `build.ps1` entry-point template with task completion and PSDepend
  bootstrap guidance.
- Declarative Task syntax, file-based caching, structured results,
  testability APIs, and v5 command guidance.
- `upgrading-to-v5.md` with migration and caching guidance.

### Changed

- Quick Start includes programmatic invocation alongside interactive usage.
- CI examples use splatting and GitHub Actions output formatting.
- The skill is v5-first while retaining v4 compatibility notes.
- Build-type references use declarative syntax and caching.
- PowerShellBuild-module guidance requires psake 5.0.0.
- The skill description uses the third-person trigger format.

### Removed

- Custom logging and OutputHandler guidance removed in psake v5.
