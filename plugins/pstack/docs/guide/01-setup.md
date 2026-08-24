# Set up pstack

In this page you install the plugin, pick which models pstack uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

Vendor pstack into the repo at `.claude/pstack/`, declare a one-plugin marketplace, and install it at project scope. From the repo root:

```bash
mkdir -p .claude/.claude-plugin
cat > .claude/.claude-plugin/marketplace.json <<'EOF'
{
  "name": "<repo-name>",
  "owner": { "name": "<you>" },
  "plugins": [{ "name": "pstack", "source": "./pstack" }]
}
EOF

claude plugin marketplace add ./.claude
claude plugin install pstack@<repo-name> --scope project
```

Name the marketplace after the repo. Marketplace names are global, so `local` collides as soon as a second repo vendors pstack.

`claude plugin validate ./.claude/pstack` checks the manifest, and `/plugin` lists what loaded. To try it without installing anything, start a session with `claude --plugin-dir ./.claude/pstack` instead.

Installed skills are namespaced, so the commands are `/pstack:poteto-mode`, `/pstack:how`, and so on. A `--plugin-dir` session and a project-scope install both put them in the `/` menu.

## Pick your models

Run:

```text
/setup-pstack
```

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) detects the models you have access to, shows you each role (code delegates, judgment, the review panels), and asks what you want. Answer the questions. It writes `.claude/pstack-models.md`, a small rule every pstack skill reads.

You only override what you care about. A role with no line in the rule keeps the skill's default. To restore a default later, delete that role's line, or just run `/setup-pstack` again.

Set a role to `inherit` and pstack omits the subagent `model` field, so the subagent runs on your parent chat model. `inherit` is not a model of its own. For a panel role the value is a list, and one subagent runs per entry, so the list length sets the panel size. Setup also configures `swarm workers`, the default model for every `/swarm` worker unless a race names a model for each arm.

## Accept the verification offer, or don't

At the end of setup, `/setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.claude/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers when it earns its place.

After setup, start a new chat. The model rule applies to new sessions.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. The first item is always "read the Principles section". The rest are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. `/poteto-mode` is sticky. It stays on for the conversation until you opt out by saying so.

Next: [Route work through `/poteto-mode`](./02-poteto-mode.md).
