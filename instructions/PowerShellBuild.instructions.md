# PowerShellBuild Instructions

This document provides step-by-step instructions for integrating
**PowerShellBuild** into your PowerShell module projects. PowerShellBuild
provides a standardized set of build, test, and publish tasks that help ensure
consistent module structure and quality across the PowerShell community.

## Overview

PowerShellBuild supports two popular PowerShell task runners:
- **[psake](https://github.com/psake/psake)** - Requires version 4.8.0 or
  greater
- **[Invoke-Build](https://github.com/nightroman/Invoke-Build)** - Version 5.8.1
  or greater

## Prerequisites

### Required PowerShell Modules

Install the required dependencies:

```powershell
# Install PowerShellBuild and its dependencies
Install-Module -Name PowerShellBuild -Repository PSGallery -Scope CurrentUser

# For psake users
Install-Module -Name psake -MinimumVersion 4.8.0 -Repository PSGallery -Scope CurrentUser

# For Invoke-Build users
Install-Module -Name InvokeBuild -Repository PSGallery -Scope CurrentUser

# Additional recommended modules (these are automatically installed as dependencies by PowerShellBuild)
Install-Module -Name BuildHelpers -Repository PSGallery -Scope CurrentUser
Install-Module -Name Pester -MinimumVersion 5.6.1 -Repository PSGallery -Scope CurrentUser -SkipPublisherCheck
Install-Module -Name PSScriptAnalyzer -Repository PSGallery -Scope CurrentUser
Install-Module -Name platyPS -Repository PSGallery -Scope CurrentUser
```

### Project Structure Requirements

Your PowerShell module project should follow this recommended structure:

```
YourModule/
├── build.ps1                    # Build script entry point
├── psakeFile.ps1               # psake task file (for psake users)
├── .build.ps1                  # Invoke-Build task file (for Invoke-Build users)
├── requirements.psd1           # Dependencies specification
├── README.md                   # Project documentation
├── CHANGELOG.md                # Change log
├── LICENSE                     # License file
├── YourModule/                 # Module source directory
│   ├── YourModule.psd1         # Module manifest
│   ├── YourModule.psm1         # Module file
│   ├── Public/                 # Public functions
│   ├── Private/                # Private functions
│   ├── Classes/                # PowerShell classes (optional)
│   └── Enum/                   # PowerShell enums (optional)
├── tests/                      # Pester tests
│   ├── YourModule.Tests.ps1    # Module tests
│   ├── Help.Tests.ps1          # Help tests
│   ├── Manifest.Tests.ps1      # Manifest tests
│   └── ScriptAnalyzerSettings.psd1  # PSScriptAnalyzer settings
├── docs/                       # Documentation (optional)
└── Output/                     # Build output (auto-generated)
```

## Setup Instructions

### Option 1: Using psake

#### 1. Create `requirements.psd1`

```powershell
@{
    PSDependOptions  = @{
        Target = 'CurrentUser'
    }
    BuildHelpers     = '2.0.16'
    Pester           = @{
        MinimumVersion = '5.6.1'
        Parameters     = @{
            SkipPublisherCheck = $true
        }
    }
    psake            = '4.9.0'
    PSScriptAnalyzer = '1.24.0'
    platyPS          = '0.14.2'
    PowerShellBuild  = 'latest'
}
```

#### 2. Create `psakeFile.ps1`

**Basic Example:**
```powershell
# Import PowerShellBuild tasks
Import-Module PowerShellBuild -Force

properties {
    # Override default settings as needed
    $PSBPreference.Test.ScriptAnalysis.Enabled = $true
    $PSBPreference.Test.CodeCoverage.Enabled = $false
    $PSBPreference.Build.CompileModule = $false
    $PSBPreference.Help.DefaultLocale = 'en-US'
}

task default -depends Build

# Reference PowerShellBuild tasks
task Clean -FromModule PowerShellBuild
task Build -FromModule PowerShellBuild
task Test -FromModule PowerShellBuild
task Analyze -FromModule PowerShellBuild
task Pester -FromModule PowerShellBuild
task Publish -FromModule PowerShellBuild
```

**Advanced Example with Custom Tasks:**
```powershell
Import-Module PowerShellBuild -Force

properties {
    # Build settings
    $PSBPreference.Build.CompileModule = $true
    $PSBPreference.Build.CompileDirectories = @('Enum', 'Classes', 'Private', 'Public')
    $PSBPreference.Build.CopyDirectories = @('Templates', 'Data')
    $PSBPreference.Build.Exclude = @('*.Tests.ps1', '*.temp.*')

    # Test settings
    $PSBPreference.Test.ScriptAnalysis.Enabled = $true
    $PSBPreference.Test.ScriptAnalysis.FailBuildOnSeverityLevel = 'Warning'
    $PSBPreference.Test.CodeCoverage.Enabled = $true
    $PSBPreference.Test.CodeCoverage.Threshold = 0.80
    $PSBPreference.Test.OutputFormat = 'NUnitXml'
    $PSBPreference.Test.OutputFile = 'TestResults.xml'

    # Documentation settings
    $PSBPreference.Docs.RootDir = './docs'
    $PSBPreference.Help.ConvertReadMeToAboutHelp = $true

    # Publishing settings
    $PSBPreference.Publish.PSRepository = 'PSGallery'
    $PSBPreference.Publish.PSRepositoryApiKey = $env:PSGALLERY_API_KEY
}

task default -depends Build

# Add custom pre-build validation
task ValidateRequirements {
    if (-not (Get-Command git -ErrorAction SilentlyContinue)) {
        throw "Git is required but not found in PATH"
    }
    Write-Host "✓ All requirements validated" -ForegroundColor Green
}

# Reference PowerShellBuild tasks with custom dependencies
task Clean -FromModule PowerShellBuild
task Build -FromModule PowerShellBuild -depends ValidateRequirements
task Test -FromModule PowerShellBuild
task Publish -FromModule PowerShellBuild

# Add custom task for local testing
task TestLocal -depends Build {
    Import-Module ./Output/$($PSBPreference.General.ModuleName) -Force
    Write-Host "Module imported for local testing" -ForegroundColor Green
}
```

#### 3. Create `build.ps1`

```powershell
[cmdletbinding(DefaultParameterSetName = 'Task')]
param(
    # Build task(s) to execute
    [parameter(ParameterSetName = 'Task', position = 0)]
    [string[]]$Task = 'default',

    # Bootstrap dependencies
    [switch]$Bootstrap,

    # List available build tasks
    [parameter(ParameterSetName = 'Help')]
    [switch]$Help,

    # PowerShell Gallery API Key
    [string]$PSGalleryApiKey
)

$ErrorActionPreference = 'Stop'

# Bootstrap dependencies
if ($Bootstrap.IsPresent) {
    Get-PackageProvider -Name Nuget -ForceBootstrap | Out-Null
    Set-PSRepository -Name PSGallery -InstallationPolicy Trusted

    if (-not (Get-Module -Name PSDepend -ListAvailable)) {
        Install-Module -Name PSDepend -Repository PSGallery -Scope CurrentUser
    }

    Import-Module -Name PSDepend -Verbose:$false
    Invoke-PSDepend -Path './requirements.psd1' -Install -Import -Force -WarningAction SilentlyContinue
}

# Set API Key if provided
if ($PSGalleryApiKey) {
    $galleryApiKey = ConvertTo-SecureString $PSGalleryApiKey -AsPlainText -Force
}

# Execute psake task(s)
$psakeFile = './psakeFile.ps1'
if ($PSCmdlet.ParameterSetName -eq 'Help') {
    Get-PSakeScriptTasks -buildFile $psakeFile | Format-Table -Property Name, Description
} else {
    Set-BuildEnvironment -Force
    Invoke-psake -buildFile $psakeFile -taskList $Task -Verbose:$VerbosePreference
    exit ([int](-not $psake.build_success))
}
```

### Option 2: Using Invoke-Build

#### 1. Create `requirements.psd1` (same as psake)

#### 2. Create `.build.ps1`

**Basic Example:**
```powershell
# Import PowerShellBuild module and tasks
Import-Module PowerShellBuild -Force
. PowerShellBuild.IB.Tasks

# Override build settings
$PSBPreference.Test.ScriptAnalysis.Enabled = $true
$PSBPreference.Test.CodeCoverage.Enabled = $false
$PSBPreference.Build.CompileModule = $false

# Default task
task . Build
```

**Advanced Example:**
```powershell
Import-Module PowerShellBuild -Force
. PowerShellBuild.IB.Tasks

# Build configuration
$PSBPreference.Build.CompileModule = $true
$PSBPreference.Build.CompileDirectories = @('Classes', 'Private', 'Public')
$PSBPreference.Build.CompileHeader = "# Generated module file`n"
$PSBPreference.Build.CompileFooter = "`n# End of generated content"

# Test configuration
$PSBPreference.Test.ScriptAnalysis.Enabled = $true
$PSBPreference.Test.CodeCoverage.Enabled = $true
$PSBPreference.Test.CodeCoverage.Threshold = 0.75
$PSBPreference.Test.OutputFile = 'TestResults.xml'

# Publishing
$PSBPreference.Publish.PSRepositoryApiKey = $env:PSGALLERY_API_KEY

# Custom tasks
task ValidateGitStatus {
    $status = git status --porcelain
    if ($status) {
        throw "Working directory is not clean. Please commit or stash changes."
    }
}

task UpdateVersion {
    # Custom version bumping logic here
    Write-Host "Version updated" -ForegroundColor Green
}

# Override task dependencies
task PublishWithValidation -Depends ValidateGitStatus, UpdateVersion, Publish

# Default task
task . Build
```

#### 3. Create `build.ps1` (similar to psake but using Invoke-Build)

```powershell
[cmdletbinding(DefaultParameterSetName = 'Task')]
param(
    [parameter(ParameterSetName = 'Task', position = 0)]
    [string[]]$Task = '.',

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

if ($PSCmdlet.ParameterSetName -eq 'Help') {
    Invoke-Build ?
} else {
    Set-BuildEnvironment -Force
    Invoke-Build -Task $Task -File ./.build.ps1
}
```

## Common Usage Patterns

### Basic Development Workflow

```powershell
# Bootstrap the project (first time only)
./build.ps1 -Bootstrap

# Build the module
./build.ps1 Build

# Run tests
./build.ps1 Test

# Run only static analysis
./build.ps1 Analyze

# Run only Pester tests
./build.ps1 Pester

# Clean output directory
./build.ps1 Clean

# Publish to PowerShell Gallery (requires API key)
$env:PSGALLERY_API_KEY = "your-api-key"
./build.ps1 Publish
```

### CI/CD Integration

#### GitHub Actions Example

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: windows-latest
    steps:
    - uses: actions/checkout@v3

    - name: Test
      shell: pwsh
      run: ./build.ps1 -Task Test -Bootstrap

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results
        path: TestResults.xml

  publish:
    needs: test
    runs-on: windows-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v3

    - name: Publish
      shell: pwsh
      run: ./build.ps1 -Task Publish -Bootstrap
      env:
        PSGALLERY_API_KEY: ${{ secrets.PSGALLERY_API_KEY }}
```

## Configuration Reference

### Build Settings (`$PSBPreference.Build.*`)

| Setting | Default | Description |
|---------|---------|-------------|
| `CompileModule` | `$false` | Compile module into single PSM1 file |
| `CompileDirectories` | `@('Enum', 'Classes', 'Private', 'Public')` | Directories to compile |
| `CopyDirectories` | `@()` | Directories to copy as-is |
| `Exclude` | `@()` | Files/patterns to exclude from build |
| `CompileHeader` | `''` | Header text for compiled module |
| `CompileFooter` | `''` | Footer text for compiled module |
| `OutDir` | `./Output` | Build output directory |

### Test Settings (`$PSBPreference.Test.*`)

| Setting | Default | Description |
|---------|---------|-------------|
| `Enabled` | `$true` | Enable Pester tests |
| `RootDir` | `./tests` | Test directory |
| `OutputFile` | `$null` | Test results output file |
| `OutputFormat` | `'NUnitXml'` | Test results format |
| `ImportModule` | `$false` | Import built module before tests |
| `ScriptAnalysis.Enabled` | `$true` | Enable PSScriptAnalyzer |
| `ScriptAnalysis.FailBuildOnSeverityLevel` | `'Error'` | Fail build threshold |
| `CodeCoverage.Enabled` | `$false` | Enable code coverage |
| `CodeCoverage.Threshold` | `0.75` | Code coverage threshold |

### Documentation Settings (`$PSBPreference.Help.*`)

| Setting | Default | Description |
|---------|---------|-------------|
| `DefaultLocale` | `(Get-UICulture).Name` | Default help locale |
| `ConvertReadMeToAboutHelp` | `$false` | Convert README to about help |

### Publishing Settings (`$PSBPreference.Publish.*`)

| Setting | Default | Description |
|---------|---------|-------------|
| `PSRepository` | `'PSGallery'` | Target repository |
| `PSRepositoryApiKey` | `$env:PSGALLERY_API_KEY` | API key for publishing |

## Available Tasks

### Primary Tasks

- **Init** - Initialize build environment
- **Clean** - Clean output directory
- **Build** - Build module (includes StageFiles, BuildHelp)
- **Analyze** - Run PSScriptAnalyzer
- **Pester** - Run Pester tests
- **Test** - Run all tests (Analyze + Pester)
- **Publish** - Publish module to repository

### Secondary Tasks

- **StageFiles** - Copy source files to output
- **BuildHelp** - Build help documentation
- **GenerateMarkdown** - Generate markdown help
- **GenerateMAML** - Generate MAML help
- **GenerateUpdatableHelp** - Generate updatable help CAB

## Troubleshooting

### Common Issues

1. **"Cannot find module 'PowerShellBuild'"**
   - Run: `Install-Module PowerShellBuild -Repository PSGallery`

2. **Build fails with permission errors**
   - Run PowerShell as Administrator or use `-Scope CurrentUser`

3. **Tests fail with module import errors**
   - Set `$PSBPreference.Test.ImportModule = $true`

4. **PSScriptAnalyzer violations fail build**
   - Fix violations or adjust `$PSBPreference.Test.ScriptAnalysis.FailBuildOnSeverityLevel`

5. **Help generation fails**
   - Ensure all functions have proper comment-based help
   - Install: `Install-Module platyPS`

### Getting Help

- Review the [PowerShellBuild documentation](https://github.com/psake/PowerShellBuild)
- Visit [https://psake.dev](https://psake.dev) for comprehensive documentation, including:
  - API reference and usage examples
  - LLM-friendly documentation formats (`llms.txt` and `llms-full.txt`)
  - Community guides and best practices
- Check existing issues on GitHub
- Look at the [test module example](https://github.com/psake/PowerShellBuild/tree/main/tests/TestModule)

## Best Practices

1. **Always use version control** - PowerShellBuild works best with git
2. **Start with basic configuration** - Add complexity gradually
3. **Use semantic versioning** - Follow [SemVer](https://semver.org/) guidelines
4. **Write comprehensive tests** - Aim for good code coverage
5. **Document your functions** - Use comment-based help
6. **Follow PowerShell naming conventions** - Use approved verbs
7. **Test locally before CI/CD** - Run `./build.ps1 Test` before pushing
8. **Keep dependencies minimal** - Only include what you actually need
9. **Use PSScriptAnalyzer rules** - Follow PowerShell best practices
10. **Version your build configurations** - Commit task files to source control

---

For more examples and advanced usage, see the PowerShellBuild [GitHub repository](https://github.com/psake/PowerShellBuild).
