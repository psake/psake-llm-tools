# Skill Changes Pull Request

## Plugin(s) Changed

<!-- Which plugin(s) does this PR affect? psake / powershellbuild / both -->

## Changes Made

<!-- Describe the changes you've made -->

## Checklist

Review the [TESTING.md](../blob/main/TESTING.md) guide before submitting.

### Skill Structure Validation

- [ ] `SKILL.md` exists and has valid YAML frontmatter (name + description)
- [ ] `references/` directory structure is intact (if applicable)
- [ ] All markdown files use proper formatting
- [ ] No broken internal references between files
- [ ] SKILL.md is under 500 lines; reference files under 300 lines each

### Content Quality

- [ ] Code examples are tested and working
- [ ] Documentation is clear and follows existing style
- [ ] Changes maintain consistency with existing content

### Testing

- [ ] Tested the skill manually in Claude (upload `.skill` or install via marketplace)
- [ ] Verified skill triggers on expected prompts
- [ ] Verified skill produces correct output for at least one scenario

## Additional Notes

<!-- Any breaking changes, migration notes, or context for reviewers -->
