# The GitOps Loop

How a change actually travels from your keyboard to a running system.
The loop has the same shape whether the target is Kubernetes, a fleet
of servers, or one Raspberry Pi.

## The stages

```
  you edit files
        |
        v
  commit + push to git
        |
        v
  review / automated checks   <-- pull request, CI, a second pair of eyes
        |
        v
  merge to the tracked branch
        |
        v
  agent notices the new commit (poll or webhook)
        |
        v
  agent applies the desired state
        |
        v
  agent verifies and keeps watching  <-- reconciliation, forever
```

## Push vs. pull

Two ways to get the change from git to the system:

**Pull (the GitOps default).** An agent *inside* the environment
watches the repo and applies changes itself. The repo never needs
credentials *into* your systems — the systems reach *out* to the repo.
New machine? Install the agent, point it at the repo, done.

**Push.** A pipeline (CI job, script) takes the new commit and pushes
changes *into* the environment — SSH, API calls, `kubectl apply` from
outside. Simpler to start, but the pipeline needs credentials for
every target, and there's usually no ongoing reconciliation.

Pull scales better and matches the principles more closely. Push is
fine for one box and an afternoon project. Know which you're doing.

## What "the agent" can be

The word sounds grand; the reality is ordinary:

- ArgoCD or Flux (Kubernetes).
- Ansible run on a schedule, pulling playbooks from git.
- A cron job running `git pull && ./apply.sh` on a server.
- A systemd timer syncing dotfiles from a repo.

The mechanism varies; the loop doesn't. If something watches a repo
and converges the machine toward it, you're doing GitOps.

## Where humans still belong

- **Writing and reviewing the change.** Automation applies; humans
  decide.
- **Approving promotion** between environments (dev → staging → prod).
- **Handling what the loop can't:** data migrations, incident
  response, anything needing judgment mid-flight.
- **Watching the watcher.** The agent itself needs monitoring — a dead
  agent means changes silently stop applying.

## A minimal example

On a single server, the whole loop can be:

```bash
# /usr/local/bin/sync-config.sh, run every 5 minutes by cron
cd /srv/desired-state
git pull --quiet
cp nginx/nginx.conf /etc/nginx/nginx.conf
systemctl reload nginx
```

Crude, no drift detection, but recognizable: repo is truth, machine
catches up, on a loop. Everything fancier is this idea with better
tooling.

Next: [GitOps Without Kubernetes](gitops-without-kubernetes.md).

## Sources

- [OpenGitOps principles](https://opengitops.dev/)
- [ArgoCD core concepts](https://argo-cd.readthedocs.io/en/stable/core_concepts/)
