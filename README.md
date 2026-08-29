# psake-tools

A collection of Claude skills for the [psake](https://psake.dev) PowerShell build automation ecosystem.
Contributions are welcome!

## Skills

| Skill | Description |
|-------|-------------|
| **psake** | Task-based build automation for .NET, Node.js, Docker, and more |
| **PowerShellBuild** | Standardized build/test/publish tasks for PowerShell module development |

## Versioning and changelogs

The marketplace catalog and each skill version independently. Update
`.claude-plugin/marketplace.json` `metadata.version` only when the catalog
schema changes. Update a plugin's own `version` when that skill changes.

- [Marketplace changelog](CHANGELOG.md) records catalog changes.
- [psake changelog](plugins/psake/CHANGELOG.md) records psake skill changes.
- [PowerShellBuild changelog](plugins/powershellbuild/CHANGELOG.md) records
  PowerShellBuild skill changes.

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
