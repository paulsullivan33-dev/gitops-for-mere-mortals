# Troubleshooting

GitOps failures have a particular flavor: the automation did exactly
what the repo said, and the repo said the wrong thing. Diagnose
repo-first.

## "The change didn't apply"

- **Agent hasn't noticed yet.** Polling intervals mean minutes of lag;
  webhooks can fail silently. Check the agent's last sync time before
  anything else.
- **Wrong branch or path.** The agent watches a specific branch and
  folder. A perfect commit to `main` does nothing if the agent tracks
  `prod`.
- **Agent is down.** The watcher needs watching — check its health
  (see [Secrets and Safety](secrets-and-safety.md)).
- **Sync is failing.** Read the agent's logs: permission errors,
  invalid syntax, and unreachable targets show up there, not in git.

## "It applied, but the system is wrong"

The repo is the truth, so the repo is wrong. Read the committed files
— not what you *think* you committed. `git log` and `git diff` are
the diagnostic tools.

- **Typo in the desired state.** A wrong port, a misspelled key.
  Fix in git, let the loop re-apply.
- **Stale value.** Someone updated the system by hand and the repo
  still describes the old intent — or vice versa. Decide which is
  correct, update the repo, let reconciliation converge.
- **Template rendered wrong.** With base+overlay or templating, check
  the *rendered* output, not the template. Most tools have a
  dry-run/diff mode — use it.

## "Drift keeps coming back"

Something outside the loop is changing the system: a hand-editing
human, another automation, or the application itself writing config
at runtime.

1. Find the writer (audit logs, file timestamps).
2. Either bring it into the repo (make it desired state) or stop it.
3. If it's the application itself (self-modifying configs), exclude
   those paths from reconciliation — fighting the app is unwinnable.

## "Rollback didn't fix it"

`git revert` restores the *description*, but some changes aren't
reversible by re-applying: data migrations, deleted data, expired
certs. The loop converges *configuration*, not history. Have a
separate plan for stateful changes — the repo can't un-migrate a
database.

## "It worked on my machine"

The local test didn't match the target: different versions, missing
dependencies, environment-specific values. Fix by making the test
environment match (containers help) and by keeping
environment-specific values in overlays, not in shared base config.

## Debugging habits

- **Repo first, machine second.** The machine is a consequence of the
  repo; start at the cause.
- **Check rendered output.** Templates lie; rendered manifests don't.
- **One change, one sync.** Same as 3D printing: shotgun edits teach
  nothing.
- **Read the agent's view.** ArgoCD's UI, `ansible --check`,
  `terraform plan` — see what the automation *thinks* before it acts.

Related: [The GitOps Loop](the-gitops-loop.md),
[Glossary](glossary.md).
