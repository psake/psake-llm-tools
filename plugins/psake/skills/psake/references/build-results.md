# Build Results and Output Formats

## `PsakeBuildResult`

`Invoke-psake -Quiet` returns one `PsakeBuildResult`. Keep it local; do not write the object to the pipeline.

| Property | Type | Meaning |
|---|---|---|
| `Success` | `bool` | Whether the build completed successfully. |
| `BuildFile` | `string` | The executed build-file path. |
| `Duration` | `TimeSpan` | Total build duration. |
| `Tasks` | `PsakeTaskResult[]` | Task outcomes in execution order. |
| `ErrorMessage` | `string` | Formatted message for the most recent build failure. |
| `ErrorRecord` | `ErrorRecord[]` | Error records for targeted diagnostics. |
| `StartedAt` / `CompletedAt` | `DateTime` | Build start and completion timestamps. |

On a normal failure, emit only `ErrorMessage`:

```powershell
$result = Invoke-psake -buildFile ./psakefile.ps1 -Quiet
if (-not $result.Success) {
    [Console]::Error.WriteLine($result.ErrorMessage)
    exit 1
}
```

For an explicit diagnostic escalation, emit only the needed detail rather than the error-record array:

```powershell
$result.ErrorRecord | ForEach-Object { [Console]::Error.WriteLine($_.ScriptStackTrace) }
```

## `PsakeTaskResult`

Each `Tasks` entry describes one task in execution order.

| Property | Type | Meaning |
|---|---|---|
| `Name` | `string` | Task name. |
| `Status` | `TaskStatus` | `Executed`, `Cached`, `Skipped`, or `Failed`. |
| `Duration` | `TimeSpan` | Task duration. |
| `Cached` | `bool` | Whether the task was served from the psake cache. |
| `ErrorMessage` | `string` | Failure message for that task, if any. |
| `ErrorRecord` | `ErrorRecord[]` | Diagnostic error records for that task, if any. |
| `InputHash` | `string` | Cache input hash, when caching applies. |

Project only the required fields:

```powershell
$taskSummary = $result.Tasks | Select-Object Name, Status, Cached
```

## Output formats

Choose a format for its consumer, not because it is merely structured:

| Consumer | Invocation | Return contract |
|---|---|---|
| LLM or ordinary automation | `Invoke-psake -Quiet` | `PsakeBuildResult` |
| Explicit complete machine-readable report | `Invoke-psake -OutputFormat JSON` | One JSON string containing the full build report |
| GitHub workflow annotations | `Invoke-psake -OutputFormat GitHubActions` | `PsakeBuildResult` plus GitHub annotation output |
| VS Code problem matcher | `Invoke-psake -OutputFormat Annotated` | `PsakeBuildResult` plus console and annotation output |
| Human terminal | `Invoke-psake` | `PsakeBuildResult` plus formatted console output |

`JSON` is intentionally complete and can consume many tokens. Use it only when the caller explicitly requests the full report. It does not return a `PsakeBuildResult`, so parse it before reading properties:

```powershell
$report = Invoke-psake -OutputFormat JSON | ConvertFrom-Json
if (-not $report.Success) { exit 1 }
```

`-Quiet` suppresses psake-generated output. A build entry point can still emit host output itself, so it must keep its captured result local to be quiet-capable.
