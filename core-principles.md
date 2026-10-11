# Core Principles

Four principles define GitOps (from the OpenGitOps project). Each is
stated plainly, then grounded in an example.

## 1. Declarative: describe the *what*, not the *how*

You write down the desired end state — "nginx version 1.25, port 80
open, this config file" — not the sequence of commands to get there.
The tooling figures out the steps.

*Example:* a file saying "3 copies of the web app, image v2.4" instead
of a runbook saying "SSH in, pull the image, stop the old container,
start three new ones..."

**Why it matters:** a description can be compared against reality. A
list of commands can't — once run, there's nothing to check.

## 2. Versioned and immutable: the description lives in git

The desired state is stored in version control. Every change is a
commit: authored, timestamped, reviewable, revertible. Nothing is
"the current state is whatever's in someone's head."

*Example:* the firewall rules for your servers live in
`firewall/rules.yaml` in a repo. Changing a rule is a pull request,
not a command typed at 2 AM.

**Why it matters:** history, blame, and rollback come free with git.
"Put it back how it was Tuesday" is a real operation.

## 3. Pulled automatically: software applies the state

An agent — not a human running commands — reads the desired state from
git and applies it to the system. Humans change the repo; machines
change the systems.

*Example:* an agent on each server watches the repo; when the config
commit changes, it pulls and reloads the service. Nobody SSHes in.

**Why it matters:** removes the human as a deployment step — and with
it, the typos, the forgotten steps, the "I thought you did that."

## 4. Continuously reconciled: reality is kept matching

The agent doesn't apply once and stop. It keeps comparing actual state
to desired state and corrects drift — or at minimum, reports it loudly.

*Example:* someone hand-edits a config on the server. The next
reconciliation notices the file no longer matches the repo and
restores it (or opens an alert, depending on how strict you configured
it).

**Why it matters:** this is the principle that separates GitOps from
"we run a deploy script from git." The loop never closes otherwise,
and drift accumulates silently.

## How the principles fit together

| Principle | Question it answers |
|---|---|
| Declarative | *What* should the system look like? |
| Versioned | *Who* changed it, *when*, and can we undo it? |
| Pulled automatically | *How* does the change reach the system? |
| Reconciled | How do we know it *stayed* that way? |

Miss any one and you have something useful, but not GitOps. A deploy
script from git (1–3, no 4) still lets hand-edits rot. A wiki page
describing the setup (1, no 2–4) is documentation, not operations.

Next: [The GitOps Loop](the-gitops-loop.md).

## Sources

- [OpenGitOps principles](https://opengitops.dev/)
