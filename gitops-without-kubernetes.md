# GitOps Without Kubernetes

GitOps is usually taught with Kubernetes, but the pattern is older
and broader: desired state in git, automation converging reality.
Here's what it looks like without a cluster in sight.

## Why this matters

If your workplace teaches GitOps as "Kubernetes plus Terraform," it's
easy to conclude it doesn't apply to your servers, your homelab, or
your dotfiles. It does. The principles don't mention containers.

## Servers and fleets

The classic non-K8s setup: a repo holds Ansible playbooks (or plain
scripts) describing each machine's configuration. A scheduled run —
cron, a CI pipeline, Ansible Tower/AWX — pulls the repo and applies
it. Hand-edits get overwritten on the next run, which is the point.

What lives in the repo:

- Package lists and versions.
- Service configs (nginx, SSH, firewall rules).
- User accounts and SSH keys.
- Cron jobs and timers.

This is how many small teams run a dozen servers sanely: the fleet's
configuration is a git repo you can read, review, and revert.

## Dotfiles and personal machines

Your own configuration — shell, editor, git settings — is desired
state for your machines. Tools like **chezmoi** or **yadm** keep
dotfiles in git and apply them per-machine, with templates for the
differences (work laptop vs. home server). New machine setup becomes
"install the tool, point it at the repo" instead of an afternoon of
remembering settings.

## What changes without Kubernetes

| With Kubernetes | Without it |
|---|---|
| Manifests (YAML) describe state | Playbooks, scripts, config files describe state |
| ArgoCD/Flux reconcile | Cron, CI, or config management reconciles |
| `kubectl diff` shows drift | Dry runs (`ansible --check`) show drift |
| Rollback is a revert + sync | Rollback is a revert + re-run |

The loop is the same; only the vocabulary changes.

## What doesn't fit as well

Kubernetes gives you a lot for free: a uniform API, built-in
diffing, mature agents. Without it, you assemble more yourself —
drift detection especially tends to be "re-run and see what changes"
rather than continuous. That's fine for many setups; be honest about
the gap rather than pretending a cron job is ArgoCD.

## Start small

Pick one thing that's currently snowflake configuration — the nginx
config on one server, your dotfiles — put it in git, and automate its
application. One loop, working, teaches more than a grand design.

Next: [Tools Overview](tools-overview.md).
