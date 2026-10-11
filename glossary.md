# Glossary

Quick definitions, A to Z. Terms point to the guide that explains
them in context.

**Agent** — software that watches the repo and applies desired state.
ArgoCD, Flux, ansible-pull, a cron job. See
[The GitOps Loop](the-gitops-loop.md).

**ArgoCD** — widely used Kubernetes GitOps agent with a web UI.

**Base + overlay** — shared configuration plus per-environment
differences. See [Common Patterns](common-patterns.md).

**Break-glass** — documented emergency procedure for intervening by
hand when automation is the problem.

**CI** — continuous integration: automated checks (tests, lint) run
on proposed changes.

**Declarative** — describing the desired end state, not the steps to
get there. See [Core Principles](core-principles.md).

**Desired state** — how the system *should* look, as stored in git.
The thing everything converges toward.

**Drift** — reality differing from desired state. Found by
reconciliation.

**Dry run** — previewing what the automation *would* change, without
changing anything (`ansible --check`, `terraform plan`).

**Flux** — Kubernetes GitOps agent, CLI-driven. Alternative to ArgoCD.

**Infrastructure as code (IaC)** — managing infrastructure with
versioned files. Adjacent to GitOps; Terraform is IaC, not GitOps by
itself.

**Kustomize** — Kubernetes-native way to do base + overlays.

**Manifest** — a file describing desired state (usually YAML in
Kubernetes land).

**OpenTofu** — open-source Terraform fork.

**Operator** — a Kubernetes agent encoding operational knowledge;
often managed via GitOps, not the same thing.

**Promotion** — moving a tested change from one environment to the
next (dev → staging → prod).

**Pull model** — agents inside the environment fetch and apply state.
The GitOps default.

**Push model** — an outside pipeline pushes changes into the
environment. Simpler; no built-in reconciliation.

**Reconciliation** — the continuous compare-and-correct loop between
desired and actual state. See [Core Principles](core-principles.md).

**Sealed Secrets** — encrypted Kubernetes secrets safe to store in
git.

**SOPS** — file encryption for secrets in git, decrypted at apply
time.

**Sync** — one application of desired state to the environment.

**Terraform** — declarative cloud infrastructure tool (push-based).
