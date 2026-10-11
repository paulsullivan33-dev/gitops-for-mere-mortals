# Secrets and Safety

The hard parts of GitOps, stated plainly. Get this wrong and the
automation faithfully deploys your mistakes — including your
passwords, to everyone with repo access.

## Secrets don't go in git. Ever.

API keys, passwords, private keys, tokens: **never committed**, even
to a private repo. "Private" is one misconfigured permission, one
future open-sourcing, one stolen laptop away from public. Git history
never forgets — removing a committed secret means rotating it, not
just deleting the file.

## What to do instead

The pattern: commit an *encrypted* secret or a *reference* to one,
and let the target environment decrypt or fetch it at apply time.

- **SOPS** (with age or PGP keys): encrypts secret files so they can
  live in git safely; the agent decrypts with its key. Simple,
  file-based, works anywhere.
- **Sealed Secrets** (Kubernetes): encrypted blobs in git that only
  the cluster can decrypt.
- **External Secrets / vault lookup** (Kubernetes): the repo holds a
  *reference* ("fetch `db-password` from the vault"); the agent
  retrieves the actual value at sync time. Nothing secret touches git
  at all.
- **Environment-provided** (servers): the agent reads secrets from the
  machine's environment or a local vault, never from the repo.

Pick one per setup. "We'll figure out secrets later" is how they end
up in git.

## Safety practices

- **Review like it deploys.** Every merge to the tracked branch *is* a
  deployment. Require review on that branch, same as production code.
- **Start with dry runs.** Ansible's `--check`, Terraform's `plan`,
  ArgoCD's diff view — look before the automation touches anything.
- **Stage promotion.** dev → staging → prod as separate repos, folders,
  or branches, with human approval between. See
  [Common Patterns](common-patterns.md).
- **Limit the agent's power.** The agent's credentials should do
  exactly its job and nothing more. A compromised agent with admin
  everywhere is the nightmare scenario — scope it down.
- **Monitor the watcher.** If the agent dies, changes silently stop
  applying and drift accumulates unnoticed. Alert on agent health,
  not just on the systems it manages.
- **Have a break-glass path.** When the automation is the problem,
  you need a documented way to intervene by hand — and a plan to
  re-converge afterward.

## What can go wrong (so you plan for it)

| Failure | Consequence | Mitigation |
|---|---|---|
| Bad commit merged | Bad state deployed everywhere, fast | Review, CI checks, staged promotion |
| Agent compromised | Attacker has the agent's privileges | Minimal credentials, short-lived tokens |
| Repo unavailable | Deployments freeze (pull model) | Highly available git hosting; cached last-known state |
| Secret committed | Credential exposure | Pre-commit hooks (gitleaks), rotation runbooks |
| Drift ignored | Hand-edits accumulate silently | Alerts on drift, regular audits |

Next: [Common Patterns](common-patterns.md).

## Sources

- [SOPS](https://github.com/getsops/sops)
- [gitleaks](https://github.com/gitleaks/gitleaks)
