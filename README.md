# psake LLM Tools

This will be a collection of tools/files that can be used with different LLM
agents. Contributions are welcome!

## psake Skill

A Claude skill for [psake](https://psake.dev), the PowerShell build automation tool.

## Installation

Download `psake.skill` from the [releases page](https://github.com/psake/psake-llm-tools/releases) and upload it to Claude.

### What's Included

- **SKILL.md** - Core psake patterns, commands, and troubleshooting
- **references/powershell-modules.md** - PowerShellBuild module for PS module development
- **references/build-types.md** - .NET, Node.js, Docker build patterns
- **references/advanced.md** - Dynamic tasks, custom logging, CI/CD integration

### Usage Examples

Ask Claude to:

- "Create a psakefile for my PowerShell module with Pester tests"
- "Help me set up a psake build for my .NET solution"
- "Generate a psakefile that creates tasks dynamically from a config file"
- "Add CI/CD integration to my existing psakefile"

### Development

#### Running Tests

There's not a good framework for testing skills yet. For now, please follow the
instructions in [testing document](TESTING.md).

## License

MIT
