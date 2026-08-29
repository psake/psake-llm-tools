# psake-tools

A collection of Claude skills for the [psake](https://psake.dev) PowerShell build automation ecosystem.
Contributions are welcome!

## Skills

| Skill | Description |
|-------|-------------|
| **psake** | Task-based build automation for .NET, Node.js, Docker, and more |
| **PowerShellBuild** | Standardized build/test/publish tasks for PowerShell module development |

## Installation

### Marketplace (recommended)

Add the marketplace in Claude Code, then install whichever skills you need:

```
/plugin marketplace add psake/psake-llm-tools
```

Once added, install individual skills with `/plugin install`.

### VSCode

You can use this link to add the marketplace (copy and paste into address bar): `vscode://chat-plugin/install?source=https://github.com/psake/psake-llm-tools/`

### Direct download (.skill files)

Download a `.skill` file from the [releases page](https://github.com/psake/psake-llm-tools/releases)
and upload it directly to Claude. Both installation methods are supported and kept in sync.

## psake Skill

### What's Included

- **SKILL.md** — Core psake patterns, commands, and troubleshooting
- **references/powershell-modules.md** — PowerShellBuild module for PS module development
- **references/build-types.md** — .NET, Node.js, Docker build patterns
- **references/advanced.md** — Dynamic tasks, custom logging, CI/CD integration

### Usage Examples

Ask Claude to:

- "Create a psakefile for my PowerShell module with Pester tests"
- "Help me set up a psake build for my .NET solution"
- "Generate a psakefile that creates tasks dynamically from a config file"
- "Add CI/CD integration to my existing psakefile"

## PowerShellBuild Skill

### What's Included

- **SKILL.md** — PSBPreference configuration, task reference, complete project examples, CI/CD patterns

### Usage Examples

Ask Claude to:

- "Set up PowerShellBuild for my PowerShell module"
- "Configure code coverage thresholds in my PowerShellBuild project"
- "Help me publish my module to PSGallery using PowerShellBuild"
- "Generate a psakeFile.ps1 using PowerShellBuild tasks"

## Development

### Testing locally

Clone the repo, then add the marketplace using a local path:

```
/plugin marketplace add ./
```

> Note: `./` is required — `.` alone will not work.

Install individual plugins to test them:

```
/plugin install psake@psake-tools
/plugin install powershellbuild@psake-tools
```

### Running Tests

Follow the instructions in [TESTING.md](TESTING.md).


### Releasing skill packages

To release one skill, update that skill's `version` in the `plugins` array of
`.claude-plugin/marketplace.json`, then merge the change to `main`. The
release workflow compares each plugin version with the preceding commit and
creates a release only for the skills whose versions changed.

For example, update the `psake` plugin version from `1.0.0` to `1.1.0` to
publish `psake-v1.1.0` with `psake.skill`. Updating both plugin versions
creates one release for each skill. The marketplace catalog `metadata.version`
changes only when its schema changes and never triggers a skill release.

### Repository Structure

```
psake-llm-tools/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace definition
├── plugins/
│   ├── psake/
│   │   └── skills/psake/       # psake skill source
│   └── powershellbuild/
│       └── skills/powershellbuild/  # PowerShellBuild skill source
└── .github/
    └── workflows/              # CI: builds .skill release packages
```

## License

MIT
