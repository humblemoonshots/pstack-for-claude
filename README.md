# pstack-for-claude

A Claude Code marketplace holding a fork of [poteto](https://x.com/poteto)'s
[pstack](https://github.com/cursor/plugins/tree/main/pstack), ported from
Cursor to Claude Code. Register it once at user level and every project gets
the plugin, rather than copying it per repo.

MIT. The root `LICENSE` covers the whole repo; `plugins/pstack/LICENSE` is
upstream's original notice, kept verbatim.

## Layout

```
.claude-plugin/marketplace.json   the catalogue
plugins/pstack/                   the plugin (upstream pstack + the port below)
```

## Install

```sh
claude plugin marketplace add humblemoonshots/pstack-for-claude
claude plugin install pstack@pstack-for-claude
```

Or declare it in `~/.claude/settings.json` so it survives a fresh machine:

```json
{
  "enabledPlugins": { "pstack@pstack-for-claude": true },
  "extraKnownMarketplaces": {
    "pstack-for-claude": {
      "source": { "source": "github", "repo": "humblemoonshots/pstack-for-claude" }
    }
  }
}
```

To hack on the plugin itself, clone the repo and point the marketplace at the
working tree instead. Claude Code then reads the tree in place, so an edit is
live in the next session with no reinstall. The path must be absolute; tilde
expansion is not documented for a directory source.

```json
"source": { "source": "directory", "path": "/absolute/path/to/pstack-for-claude" }
```

## Port notes

Upstream targets Cursor. Beyond the manifest rename an earlier per-repo
vendored copy already carried (`.cursor-plugin/` → `.claude-plugin/`, documented in the plugin's own
README), this fork changes the following. Reapply the table after any upstream
sync.

| upstream (Cursor) | here (Claude Code) | why |
| --- | --- | --- |
| `disable-model-invocation: true` on the 21 `principle-*` leaves | `user-invocable: false` | On Claude Code the upstream flag keeps a skill in the `/` menu and hides it from the model — the opposite of the intent. The leaves are internal references that `poteto-mode` tells the model to read. |
| `~/.cursor/projects/<slug>/agent-transcripts` in `worktree-audit.sh` | `~/.claude/projects/<slug>/*.jsonl` | Different root, no `agent-transcripts` subdirectory, different slug rule, and Claude Code keys the directory by working directory so each worktree has its own. |
| slug = strip leading `/`, map `/` → `-` | slug = every character outside `[A-Za-z0-9-]` → `-` | The upstream rule gets the name wrong for any path with a space, `~`, `@` or `.`. |
| "run Cursor's built-in **babysit** skill" (`opening-a-pr.md`) | `playbooks/babysit.md` | No such built-in here. |
| "replaces Cursor's built-in babysit skill … do not route there" (`babysit.md`) | removed | Warns against something that does not exist here. |
| "the cloud agent's status in the Cursor dashboard" (`orchestrate.md`) | "where you launched it" | No Cursor dashboard. |

Deliberately left alone: `endCursor` (GraphQL pagination), the `author ===
"cursor"` / `CURSOR_AUTOMATION_ID` parsing in `watch-pr` (that is Cursor's PR
bot, still real if you use it), `~/Library/Application Support/Cursor` in the
disk-reclaim list, and the `@cursor-skill/` package name (cosmetic, and renaming
it would churn `bun.lock`).

The 18 command-style skills (`arena`, `tdd`, `architect`, …) keep
`disable-model-invocation: true`. They are workflows you trigger deliberately,
which is what that flag is for.

## Behavioural deviations

The table above translates Cursor mechanics into Claude Code mechanics without changing what the plugin does. The changes below do change behaviour. Each is a choice made on the port's own terms, with the reasoning here so it can be revisited. Reapply them after any upstream sync.

| skill | upstream | here | why |
| --- | --- | --- | --- |
| `technical-writing` | `disable-model-invocation: true`, so only a person can run it | model-invocable | It is guidance with no subagents and no side effects, and its description already names when it applies (docs, readmes, PR descriptions, commit messages). Workflows that write documentation should be able to invoke it by name at that point rather than stop and ask a person to. `unslop`, the same kind of skill, is already model-invocable, and this makes the two consistent. |

## Known upstream bug, not ported around

`worktree-audit.sh` parses `git worktree list --porcelain` with awk `$2`, which
truncates any worktree path containing a space. Present upstream; left as-is so
it stays a clean diff against upstream.

## Syncing upstream

Upstream was at 0.14.2 when this fork was cut from 0.14.1.

```sh
# diff this tree against a fresh upstream checkout, then reapply the tables above
git clone --depth 1 --filter=blob:none --sparse https://github.com/cursor/plugins /tmp/cursor-plugins
git -C /tmp/cursor-plugins sparse-checkout set pstack
diff -ru /tmp/cursor-plugins/pstack plugins/pstack
```

After any change: `claude plugin validate .` and `claude plugin validate ./plugins/pstack/skills`.
