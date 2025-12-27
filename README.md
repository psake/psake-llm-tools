# psake Skill

A Claude skill for [psake](https://psake.dev), the PowerShell build automation tool.

## Installation

Download `psake.skill` from the [releases page](../../releases) and upload it to Claude.

## What's Included

- **SKILL.md** - Core psake patterns, commands, and troubleshooting
- **references/powershell-modules.md** - PowerShellBuild module for PS module development
- **references/build-types.md** - .NET, Node.js, Docker build patterns
- **references/advanced.md** - Dynamic tasks, custom logging, CI/CD integration

## Usage Examples

Ask Claude to:

- "Create a psakefile for my PowerShell module with Pester tests"
- "Help me set up a psake build for my .NET solution"
- "Generate a psakefile that creates tasks dynamically from a config file"
- "Add CI/CD integration to my existing psakefile"

## Development

### Running Tests

```bash
cd tests
pip install -r requirements.txt
export ANTHROPIC_API_KEY="your-key"
python run_evaluations.py --list  # See available scenarios
python run_evaluations.py         # Run all tests
```

### Packaging

```bash
# Using the skill-creator scripts (if available)
python package_skill.py . ./dist
```

## License

MIT
