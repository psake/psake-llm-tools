# Testing the psake Skill

## Prerequisites

1. **Install the skill** in Claude (claude.ai or Claude Desktop)
2. **Have PowerShell available** for syntax validation:
   - Windows: Built-in
   - macOS: `brew install powershell`
   - Linux: `sudo apt-get install powershell`

## Test Scenarios

Run each scenario in a new Claude conversation with the skill installed.

---

### Scenario 1: PowerShell Module Build

**Prompt:**

```
Create a psakefile.ps1 for a PowerShell module called 'NetworkTools' with these requirements:
- Source files are in ./src/NetworkTools/
- Tests are in ./tests/ using Pester
- Tasks needed: Clean, Build (copy to ./output/), Test (run Pester), Analyze (PSScriptAnalyzer)
- Default task should run Build and Test
- Use proper variable scoping for sharing data between tasks

After generating the psakefile, validate it.
```

**Expected output contains:**

- [ ] `Properties { }` block
- [ ] `Task Clean`
- [ ] `Task Build`
- [ ] `Task Test` with Pester invocation
- [ ] `Task Analyze` with PSScriptAnalyzer
- [ ] `Task Default -depends`
- [ ] `$script:` for shared variables (if applicable)
- [ ] Validation confirms syntax is correct

---

### Scenario 2: .NET Solution Build

**Prompt:**

```
Create a psakefile.ps1 for a .NET 8 solution with these requirements:
- Solution file is ./MyApp.sln
- Output directory is ./build/
- Tasks: Clean, Restore, Build, Test, Pack (create NuGet packages)
- Build should use Release configuration by default but be overridable
- Test task should generate test results in TRX format
- Pack should only run if all tests pass

After generating the psakefile, validate it.
```

**Expected output contains:**

- [ ] `Properties { }` with `$Configuration = 'Release'`
- [ ] `exec { dotnet restore }`
- [ ] `exec { dotnet build }` with configuration parameter
- [ ] `exec { dotnet test }` with TRX logger
- [ ] `exec { dotnet pack }`
- [ ] `-depends` chains enforcing order
- [ ] Validation confirms syntax is correct

---

### Scenario 3: File Management Automation

**Prompt:**

```
Create a psakefile.ps1 for automated file management with these requirements:
- Task: CleanOldLogs - Delete .log files older than 30 days from ./logs/
- Task: ArchiveReports - Move .xlsx files from ./reports/ to ./archive/YYYY-MM/ based on file date
- Task: BackupConfig - Copy all .json and .xml files from ./config/ to ./backup/ with timestamp
- Task: GenerateReport - Create a summary of all operations performed
- Use $script: scoping to track operations across tasks for the report

After generating the psakefile, validate it.
```

**Expected output contains:**

- [ ] `Get-ChildItem` for file discovery
- [ ] Date comparison with `LastWriteTime`
- [ ] `Remove-Item`, `Move-Item`, `Copy-Item` operations
- [ ] `$script:` variables for cross-task state
- [ ] Path construction with `Join-Path`
- [ ] Validation confirms syntax is correct

---

### Scenario 4: Dynamic Task Generation

**Prompt:**

```
Create a psakefile.ps1 that dynamically generates build tasks:
- Read a projects.json file that lists multiple projects with their paths and build types
- Generate a build task for each project (e.g., build:ProjectA, build:ProjectB)
- Generate a test task for each project
- Create a BuildAll task that depends on all generated build tasks
- Handle both 'dotnet' and 'npm' build types differently

After generating the psakefile, validate it.
```

**Expected output contains:**

- [ ] `Get-Content` and `ConvertFrom-Json`
- [ ] `foreach` loop creating tasks
- [ ] `.GetNewClosure()` for variable capture
- [ ] Dynamic `-depends` construction
- [ ] `switch` or `if` for build type handling
- [ ] Validation confirms syntax is correct

---

### Scenario 5: CI/CD Integration

**Prompt:**

```
Create a psakefile.ps1 designed for CI/CD pipelines:
- Detect if running in CI via environment variables ($env:CI, $env:GITHUB_ACTIONS)
- Task: Build with version number from $env:BUILD_NUMBER or default to '1.0.0-local'
- Task: Test that outputs results in a CI-friendly format
- Task: Publish that only runs on main branch (check $env:GITHUB_REF)
- Use -precondition for conditional task execution
- Include proper error handling with exec for all external commands

After generating the psakefile, validate it.
```

**Expected output contains:**

- [ ] `$env:CI` or `$env:GITHUB_ACTIONS` checks
- [ ] `-precondition { }` on appropriate tasks
- [ ] `exec { }` wrapping external commands
- [ ] Version defaulting logic
- [ ] Branch check for publish
- [ ] Validation confirms syntax is correct

---

## Recording Results

Use this template to record your test results:

```markdown
# psake Skill Test Results

**Date:** YYYY-MM-DD
**Skill Version:** X.X.X
**Tester:** [Name]

| Scenario | Syntax Valid | Expected Elements | Pass/Fail | Notes |
|----------|--------------|-------------------|-----------|-------|
| 1. PowerShell Module | ✅/❌ | X/7 | | |
| 2. .NET Solution | ✅/❌ | X/7 | | |
| 3. File Management | ✅/❌ | X/6 | | |
| 4. Dynamic Tasks | ✅/❌ | X/6 | | |
| 5. CI/CD Integration | ✅/❌ | X/6 | | |

## Issues Found

[Describe any problems or unexpected behavior]

## Suggested Improvements

[Note any patterns that should be added to the skill]
```
