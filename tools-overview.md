# Tools Overview

What the tools actually do, without the marketing. Pick by matching
the tool to your target — not by hype.

## Kubernetes GitOps

**ArgoCD.** The most widely used K8s GitOps agent. Watches git repos,
shows what it manages in a web UI, syncs the cluster to match, and
reports drift visually. Pull-based. If someone says "GitOps tool,"
this is usually what they mean.

**Flux.** The other major K8s GitOps agent (a CNCF project like
ArgoCD). More CLI-driven, less UI. Comparable job, different taste.
Either is a fine choice; teams usually pick by which interface they
prefer.

Honest note: both assume real Kubernetes. For a homelab learning
cluster they're great; for one VPS they're overkill.

## Configuration management (servers without K8s)

**Ansible.** Agentless: SSHes into machines and applies playbooks
(YAML describing desired state). The pull-model version runs
`ansible-pull` on a schedule from each machine. Huge module library,
readable playbooks, the default answer for "manage my servers from
git."

**Chef / Puppet.** Older, agent-based config management. Still running
large fleets reliably. Not where new projects start in 2026, but
worth knowing the names when you inherit them.

**Salt.** Fast, scalable, agent-based. A solid choice at larger scale;
more machinery than Ansible for small setups.

## Infrastructure as code (adjacent, not identical)

**Terraform / OpenTofu.** Describe cloud infrastructure (VMs,
networks, DNS) in files, apply to create it. Declarative and
versioned like GitOps — but classically *push*-based (you run apply)
with no built-in reconciliation loop. Pairs well with GitOps: Terraform
provisions the machines, GitOps configures what's on them.

**Important distinction:** "we keep Terraform in git" is good
practice, not GitOps by itself. The reconciliation loop is the
missing principle.

## Small-scale and personal

**chezmoi / yadm.** Dotfiles in git, applied per machine. GitOps for
your own configuration.

**Plain scripts + cron/systemd timers.** `git pull && ./apply.sh` on
a schedule. Unfashionable, fully functional, and the right size for
one server. Don't let anyone shame you out of what works.

## How to choose

1. **Kubernetes?** ArgoCD or Flux.
2. **Servers/VMs?** Ansible (usually `ansible-pull` for the GitOps
   shape).
3. **Cloud resources?** Terraform/OpenTofu for provisioning, plus one
   of the above for configuration.
4. **Just your machines?** chezmoi/yadm, or scripts on a timer.

One tool per job, learned well, beats a shelf of half-configured
platforms.

Next: [Secrets and Safety](secrets-and-safety.md).
