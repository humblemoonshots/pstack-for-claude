# pstack-for-claude

A Claude Code marketplace holding my fork of [poteto](https://x.com/poteto)'s
[pstack](https://github.com/cursor/plugins/tree/main/pstack). Registered once at
user level so every project gets it, rather than copying the plugin per repo.

Upstream is MIT; see `plugins/pstack/LICENSE`.

## Layout

```
.claude-plugin/marketplace.json   the catalogue
plugins/pstack/                   the plugin (upstream pstack + the port below)
```

## Install

Already wired up in `~/.claude/settings.json`:

```json
{
  "enabledPlugins": { "pstack@pstack-for-claude": true },
  "extraKnownMarketplaces": {
    "pstack-for-claude": {
      "source": { "source": "directory", "path": "/Users/merott/workspace/pstack-for-claude" }
    }
  }
}
```

`directory` is the development source type: Claude Code reads this working tree
in place, so an edit here is live in the next session with no reinstall. The
path is absolute on purpose — tilde expansion is not documented for a directory
source. If this ever stops being a tree I edit, switch to a `git` source
pointing at the private remote and let Claude Code clone it instead.

## Port notes

Upstream targets Cursor. Beyond the manifest rename the Notebox copy already
carried (`.cursor-plugin/` → `.claude-plugin/`, documented in the plugin's own
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

## Known upstream bug, not ported around

`worktree-audit.sh` parses `git worktree list --porcelain` with awk `$2`, which
truncates any worktree path containing a space. Present upstream; left as-is so
it stays a clean diff against upstream.

## Syncing upstream

Upstream was at 0.14.2 when this fork was cut from 0.14.1.

```sh
# diff this tree against a fresh upstream checkout, then reapply the table above
git clone --depth 1 --filter=blob:none --sparse https://github.com/cursor/plugins /tmp/cursor-plugins
git -C /tmp/cursor-plugins sparse-checkout set pstack
diff -ru /tmp/cursor-plugins/pstack plugins/pstack
```

After any change: `claude plugin validate .` and `claude plugin validate ./plugins/pstack/skills`.
