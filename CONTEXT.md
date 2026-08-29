# psake LLM Tools

Shared language for guidance that lets agents and humans run psake builds with intentional output levels.

## Invocation modes

**Quiet automation**:
An agent or non-annotation CI invocation using a quiet-capable build entry point. It suppresses psake-generated output while a successful run exposes a `PsakeBuildResult` for local inspection and a failed run emits only an error summary.
_Avoid_: Default invocation, silent build

**Interactive execution**:
An invocation selected to display psake's formatted build transcript for human observation.
_Avoid_: Verbose build

**Diagnostic escalation**:
A quiet invocation selected after an explicit troubleshooting request or an insufficient error summary; it emits targeted error detail rather than the full error-record array.
_Avoid_: Debug mode, verbose build

**Quiet-capable entry point**:
A project build entry point that forwards its quiet option to psake and does not emit the captured `PsakeBuildResult`.
_Avoid_: Quiet wrapper

## Report formats

**JSON report**:
The complete build report returned as a JSON string by `Invoke-psake -OutputFormat JSON`, including result and per-task detail. It is distinct from `PsakeBuildResult` and is intended for an explicit machine-readable consumer.
_Avoid_: Quiet result, agent default

**Build result model**:
The `PsakeBuildResult` aggregate returned by quiet invocation and its ordered `Tasks` collection of `PsakeTaskResult` entries. It is documented as one model because task outcomes are part of a build outcome.

**Host output**:
Output written directly to the console by a project's build entry point. It is separate from a command's returned value and is not suppressed by psake's `-Quiet` option.
_Avoid_: Return value, variable output

**Annotated output**:
`Invoke-psake -OutputFormat Annotated`, which writes console output plus VS Code problem-matcher annotation lines.
_Avoid_: GitHub Actions output

**GitHub Actions output**:
`Invoke-psake -OutputFormat GitHubActions`, which writes GitHub workflow annotation lines.
_Avoid_: Annotated output

**Consumer-specific format**:
Output-format selection based on its intended consumer: quiet automation for ordinary agents, JSON for an explicit complete report, GitHub Actions output for workflow annotations, and annotated output for VS Code problem matchers.
_Avoid_: Generic CI output

## Versioning

**Marketplace catalog version**:
The global API or schema version of the central marketplace registry, such as `marketplace.json`. It changes only when the marketplace data structure changes, not when an individual skill updates.
_Avoid_: Skill version, marketplace release version

## Failure detail

**Error summary**:
The `ErrorMessage` string from `PsakeBuildResult`, containing the most recent failure message.
_Avoid_: Short error, error object

**Error detail**:
The `ErrorRecord` array from `PsakeBuildResult`, whose error records may contain diagnostic context such as stack traces.
_Avoid_: Error, full error message
