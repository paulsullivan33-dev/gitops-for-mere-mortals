# What GitOps Is

GitOps is a way of running systems where **git holds the description
of how things should be**, and **automation makes reality match**.
You change a file, commit, push — and software applies the change for
you. If someone hand-edits the system afterward, the automation
notices the difference and puts it back.

The one-sentence version: *the repository is the truth; the machines
catch up.*

## The idea, concretely

Without GitOps, deploying usually looks like this: someone SSHes in,
runs commands, edits configs by hand. It works — until it doesn't, and
nobody can say exactly what the machine's state is, because the state
lives in someone's memory and shell history.

With GitOps:

1. You describe the desired state in files (configs, manifests,
   scripts) and commit them to git.
2. An agent — a program watching the repo — sees the new commit.
3. It applies the change to the system.
4. It keeps watching. If reality drifts from the description, it
   corrects it (or at least reports it).

The history of *what changed, when, and why* is just the git log.
Rolling back is `git revert` plus the agent doing its job.

## What GitOps isn't

- **Not a product.** It's a practice. ArgoCD and Flux are GitOps
  *tools*; GitOps itself is the pattern they implement.
- **Not only Kubernetes.** K8s is where the term got famous, but the
  pattern predates it and works without it. See
  [GitOps Without Kubernetes](gitops-without-kubernetes.md).
- **Not magic deployment.** Something still has to *apply* the state —
  an agent, a pipeline, a cron job running a script. GitOps names the
  pattern; you still build or choose the machinery.
- **Not a substitute for judgment.** Automation applies what you
  committed, including your mistakes. Review changes like they go
  straight to production — because they do.

## Why people adopt it

- **Auditability.** Every change is a commit with an author and a
  message. "Who changed this?" is `git log`, not archaeology.
- **Reproducibility.** New machine? Point the agent at the repo. The
  setup is data, not folklore.
- **Drift resistance.** Hand-edits get noticed and corrected instead
  of silently accumulating.
- **Familiar workflow.** Developers already know pull requests and
  code review. GitOps reuses that muscle for operations.

## The honest costs

- **Learning curve.** The agent, the repo layout, and the failure modes
  are new things to learn.
- **Git becomes critical infrastructure.** If the repo is down or
  wrong, deployments stop or break. Protect it accordingly.
- **Not everything fits.** Secrets, rapidly changing runtime data, and
  things that need human judgment mid-deploy don't belong in the
  desired-state repo. See [Secrets and Safety](secrets-and-safety.md).

Next: [Core Principles](core-principles.md).

## Sources

- [OpenGitOps principles](https://opengitops.dev/)
- [GitOps Principles (Weaveworks, archived)](https://www.weave.works/technologies/gitops/)
