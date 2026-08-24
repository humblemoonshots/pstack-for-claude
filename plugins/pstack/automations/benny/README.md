# benny

> **porting note.** benny was written for cursor automations. this fork repoints its paths, tool names and model names at claude code, but the two automation prompts in `templates/` still describe a cursor-side trigger. the claude code equivalent is a [routine](https://code.claude.com/docs/en/routines) created with `/schedule`; wire the prompts into one before enabling benny.

benny gives you two scheduled claude code routines for slack issue reports. one triages each report. the other reproduces confirmed bugs and may prepare a small draft fix.

the files in this directory are dormant setup and automation sources. they do not appear as slash skills.

## set it up

1. point claude code at [`FOR_AGENTS.md`](./FOR_AGENTS.md) and name the target repository.
2. let setup merge this whole directory into the target at `.claude/automations/benny/`. it must preserve destination-only files and review conflicts instead of overwriting local edits.
3. let setup enable pstack in the target repository's `.claude/settings.json` for shared dependencies:

```json
{
	"extraKnownMarketplaces": {
		"local": { "source": { "source": "directory", "path": "." } }
	},
	"enabledPlugins": {
		"pstack@local": true
	}
}
```

4. keep user-owned configuration outside the copied pack, for example in `.claude/benny/`. adapt [`configuration.example.yaml`](./templates/configuration.example.yaml) and [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md).
5. commit `.claude/settings.json`, `.claude/automations/benny/`, and any secret-free configuration before enabling either automation.
6. review each new automation draft or update existing automations in their editors. then send a harmless test report and verify every source-channel post stays in the original thread.
