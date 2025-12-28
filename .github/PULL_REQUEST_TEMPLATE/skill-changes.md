# Skill Changes Pull Request

## Changes Made

<!-- Describe the changes you've made to the psake skill -->

## Checklist

Review the TESTING.md in the repository.

### Skill Structure Validation

- [ ] `SKILL.md` exists and is properly formatted
- [ ] YAML frontmatter is present and valid (name, description)
- [ ] `references/` directory structure is intact
- [ ] All markdown files use proper formatting
- [ ] No broken internal references between files

### Content Quality

- [ ] Code examples are tested and working
- [ ] Documentation is clear and follows existing style
- [ ] Examples include necessary context and explanations
- [ ] Changes maintain consistency with existing content

### Testing

Before submitting this PR, I have tested the following scenarios (check all that apply):

- [ ] **Scenario 1: PowerShell Module Build** - Verified Claude can generate correct PowerShell module build scripts
- [ ] **Scenario 2: .NET Solution Build** - Verified .NET build task creation and configuration
- [ ] **Scenario 3: File Management Automation** - Tested file copy/clean/archive patterns
- [ ] **Scenario 4: Dynamic Task Generation** - Validated dynamic task and dependency patterns
- [ ] **Scenario 5: CI/CD Integration** - Checked GitHub Actions/Azure Pipelines examples

### Documentation

- [ ] Updated relevant sections in `SKILL.md`
- [ ] Added/updated reference files if needed
- [ ] Verified all line counts are reasonable (SKILL.md < 500 lines, references < 300 lines each)

## Additional Notes

<!-- Any additional context, breaking changes, or migration notes -->
