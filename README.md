# Thunderbird Extension Skill for Claude Code

A [Claude Code](https://claude.com/claude-code) skill for Thunderbird MailExtension development. Covers the full lifecycle: project structure, manifest configuration, API usage, Experiment APIs, debugging, building, and compatibility handling.

## What's Included

- **SKILL.md** — Main skill file with quick reference, extension structure, development workflow, common patterns, and troubleshooting
- **references/manifest-keys.md** — Every supported manifest key, MV2 vs MV3, Thunderbird-specific keys, and permission requirements
- **references/webextension-apis.md** — Complete list of supported Thunderbird-specific APIs and Firefox-compatible APIs
- **references/experiments.md** — How to write Experiment APIs (schema, parent/child implementation, lifecycle, scope handling, debugging)
- **references/version-updates.md** — Breaking changes per Thunderbird version (TB 68 through TB 151)

## Usage

Install this skill in Claude Code:

```bash
mkdir -p ~/.claude/skills/thunderbird-extension
cp -r * ~/.claude/skills/thunderbird-extension/
```

Then invoke it with `/thunderbird-extension` in Claude Code.

## References

- [Thunderbird WebExtension API Reference](https://webextension-api.thunderbird.net/)
- [Thunderbird Add-on Developer Docs](https://developer.thunderbird.net/add-ons/mailextensions)
- [Example Extensions](https://github.com/thunderbird/webext-examples)
- [Shared Experiments](https://github.com/thunderbird/webext-experiments)
- [Add-on Community Support](https://discuss.thunderbird.net/groups/addons)
