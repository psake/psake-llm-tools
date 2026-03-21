---
name: powershellbuild
description: PowerShellBuild module for standardized PowerShell module development. Use when Claude needs to help set up or troubleshoot PowerShellBuild-based projects, generate psakeFile.ps1 or .build.ps1 files using PowerShellBuild tasks, configure build/test/publish pipelines for PowerShell modules, or work with PSBPreference settings. Triggers include mentions of PowerShellBuild, PSBPreference, PowerShell module builds, Pester, PSScriptAnalyzer, PlatyPS, PSGallery publishing, or any request to set up a PowerShell module build system.
---

# PowerShellBuild

PowerShellBuild provides standardized build, test, and publish tasks for PowerShell modules. It works with both **psake** (≥ 4.8.0) and **Invoke-Build** (≥ 5.8.1).

## Decision Tree

**Which task runner are you using?**
- **psake** → use `task <Name> -FromModule PowerShellBuild` pattern
- **Invoke-Build** → use `. PowerShellBuild.IB.Tasks` pattern
- **Not sure / starting fresh** → default to psake (simpler syntax)

**What do you need?**
- Set up a new module project → [Complete Example](#complete-example)
- Override build behavior → [Configuration ($PSBPreference)](#configuration-psbpreference)
- Customize task dependencies → [Modifying Task Dependencies](#modifying-task-dependencies)
- CI/CD setup → [CI/CD Integration](#cicd-integration)

## Quick Start

```powershell
Install-Module -Name PowerShellBuild -Repository PSGallery -Scope CurrentUser
Install-Module -Name psake -MinimumVersion 4.8.0 -Repository PSGallery -Scope CurrentUser
```

### Minimal psakeFile.ps1

> **Do NOT `Import-Module PowerShellBuild` in psakeFile.ps1** — `-FromModule` loads the module automatically when psake parses the task definitions.

```powershell
properties {
    $PSBPreference.Test.ScriptAnalysis.Enabled = $true
    $PSBPreference.Test.CodeCoverage.Enabled   = $false
}

task default -depends Test

task Test    -FromModule PowerShellBuild
task Publish -FromModule PowerShellBuild
```

This gives you: `Init → Clean → StageFiles → BuildHelp → Build → Analyze → Pester → Test → Publish`

## Project Structure

```
MyModule/
├── build.ps1              # Entry point
├── psakeFile.ps1          # Build tasks (psake)
├── .build.ps1             # Build tasks (Invoke-Build)
├── requirements.psd1      # Dependencies
├── MyModule/              # Source directory
│   ├── MyModule.psd1      # Module manifest
│   ├── MyModule.psm1      # Module root
│   ├── Public/            # Exported functions
│   └── Private/           # Internal functions
├── tests/
│   └── MyModule.Tests.ps1
└── Output/                # Build output (auto-generated)
```

## Available Tasks

### Primary Tasks

| Task    | Depends On          | Description                        |
|---------|---------------------|------------------------------------|
| Init    | —                   | Initialize build environment       |
| Clean   | Init                | Remove output directory            |
| Build   | StageFiles, BuildHelp | Compile module to output          |
| Analyze | Build               | Run PSScriptAnalyzer               |
| Pester  | Build               | Run Pester tests                   |
| Test    | Analyze, Pester     | Run all quality checks             |
| Publish | Test                | Publish to PowerShell Gallery      |

### Secondary Tasks

| Task               | Description                          |
|--------------------|--------------------------------------|
| StageFiles         | Copy source files to output          |
| GenerateMarkdown   | Generate PlatyPS markdown help       |
| GenerateMAML       | Convert markdown to MAML help        |
| BuildHelp          | Run all help generation              |

## Configuration ($PSBPreference)

Set these in your `properties` block before referencing PowerShellBuild tasks.

### Build

```powershell
$PSBPreference.General.ModuleName              = 'MyModule'       # auto-detected from manifest
$PSBPreference.General.SrcRootDir              = './MyModule'     # default: project root
$PSBPreference.Build.OutDir                    = './Output'
$PSBPreference.Build.CompileModule             = $true            # merge into single PSM1
$PSBPreference.Build.CompileDirectories        = @('Enum', 'Classes', 'Private', 'Public')
$PSBPreference.Build.CopyDirectories           = @('Data')        # copy as-is (no compile)
$PSBPreference.Build.Exclude                   = @('*.Tests.ps1')
```

### Test

```powershell
$PSBPreference.Test.Enabled                                = $true
$PSBPreference.Test.RootDir                               = './tests'
$PSBPreference.Test.OutputFile                            = 'TestResults.xml'
$PSBPreference.Test.OutputFormat                          = 'NUnitXml'
$PSBPreference.Test.ScriptAnalysis.Enabled                = $true
$PSBPreference.Test.ScriptAnalysis.FailBuildOnSeverityLevel = 'Error'
$PSBPreference.Test.CodeCoverage.Enabled                  = $true
$PSBPreference.Test.CodeCoverage.Threshold                = 0.75   # 0.0–1.0
```

### Help & Docs

```powershell
$PSBPreference.Help.DefaultLocale             = 'en-US'
$PSBPreference.Help.ConvertReadMeToAboutHelp  = $false
$PSBPreference.Docs.RootDir                   = './docs'
```

### Publish

```powershell
$PSBPreference.Publish.PSRepository        = 'PSGallery'
$PSBPreference.Publish.PSRepositoryApiKey  = $env:PSGALLERY_API_KEY
```

## Modifying Task Dependencies

Set these variables **before** the `-FromModule` references take effect:

```powershell
$PSBBuildDependency   = 'StageFiles'  # skip help generation
$PSBTestDependency    = 'Pester'      # skip analysis
$PSBPublishDependency = 'Build'       # publish without tests (not recommended)
```

## Complete Example

### build.ps1

```powershell
[cmdletbinding(DefaultParameterSetName = 'Task')]
param(
    [parameter(ParameterSetName = 'Task', position = 0)]
    [string[]]$Task = 'default',

    [switch]$Bootstrap,

    [parameter(ParameterSetName = 'Help')]
    [switch]$Help
)

$ErrorActionPreference = 'Stop'

if ($Bootstrap.IsPresent) {
    Get-PackageProvider -Name Nuget -ForceBootstrap | Out-Null
    Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
    if (-not (Get-Module -Name PSDepend -ListAvailable)) {
        Install-Module -Name PSDepend -Repository PSGallery -Scope CurrentUser
    }
    Import-Module -Name PSDepend -Verbose:$false
    Invoke-PSDepend -Path './requirements.psd1' -Install -Import -Force -WarningAction SilentlyContinue
}

$psakeFile = './psakeFile.ps1'
if ($PSCmdlet.ParameterSetName -eq 'Help') {
    Get-PSakeScriptTasks -buildFile $psakeFile | Format-Table -Property Name, Description
} else {
    Set-BuildEnvironment -Force
    Invoke-psake -buildFile $psakeFile -taskList $Task -Verbose:$VerbosePreference
    exit ([int](-not $psake.build_success))
}
```

### requirements.psd1

```powershell
@{
    PSDependOptions  = @{ Target = 'CurrentUser' }
    psake            = '4.9.0'
    PowerShellBuild  = 'latest'
    Pester           = @{
        MinimumVersion = '5.6.1'
        Parameters     = @{ SkipPublisherCheck = $true }
    }
    PSScriptAnalyzer = '1.24.0'
    platyPS          = '0.14.2'
}
```

### psakeFile.ps1 (full)

```powershell
properties {
    $PSBPreference.Build.CompileModule             = $true
    $PSBPreference.Build.CompileDirectories        = @('Enum', 'Classes', 'Private', 'Public')
    $PSBPreference.Test.ScriptAnalysis.Enabled     = $true
    $PSBPreference.Test.CodeCoverage.Enabled       = $true
    $PSBPreference.Test.CodeCoverage.Threshold     = 0.80
    $PSBPreference.Publish.PSRepositoryApiKey      = $env:PSGALLERY_API_KEY
}

task default -depends Test

task Clean   -FromModule PowerShellBuild
task Build   -FromModule PowerShellBuild
task Analyze -FromModule PowerShellBuild
task Pester  -FromModule PowerShellBuild
task Test    -FromModule PowerShellBuild
task Publish -FromModule PowerShellBuild
```

## Invoke-Build Alternative

```powershell
. PowerShellBuild.IB.Tasks

$PSBPreference.Build.CompileModule         = $true
$PSBPreference.Test.CodeCoverage.Enabled   = $true
$PSBPreference.Test.CodeCoverage.Threshold = 0.75

task . Build
```

## CI/CD Integration

### GitHub Actions

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Test
        shell: pwsh
        run: ./build.ps1 -Task Test -Bootstrap

  publish:
    needs: test
    runs-on: windows-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Publish
        shell: pwsh
        run: ./build.ps1 -Task Publish -Bootstrap
        env:
          PSGALLERY_API_KEY: ${{ secrets.PSGALLERY_API_KEY }}
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Task 'Build' not found" | Ensure psake ≥ 4.8.0 for `-FromModule` support |
| Module not found | Run `./build.ps1 -Bootstrap` first |
| BuildHelp fails | Install PlatyPS: `Install-Module platyPS` |
| Tests not found | Check `$PSBPreference.Test.RootDir` matches your `tests/` path |
| ScriptAnalyzer fails build | Fix violations or set `FailBuildOnSeverityLevel = 'Warning'` |
| Code coverage below threshold | Raise `CodeCoverage.Threshold` or add tests |
| Publish fails | Verify `PSGALLERY_API_KEY` env var is set |
