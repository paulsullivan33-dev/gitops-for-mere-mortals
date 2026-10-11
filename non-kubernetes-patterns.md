# GitOps Patterns Without Kubernetes

Concrete, copyable GitOps patterns for servers, homelabs, and personal
machines — no cluster required. Each pattern follows the same shape:
the idea, the repo layout, the loop that applies it, and the honest
limits. For the concepts behind them, see
[GitOps Without Kubernetes](gitops-without-kubernetes.md).

## 1. Ansible-pull fleet (the workhorse)

The canonical non-K8s GitOps setup. A repo holds Ansible playbooks
describing every machine; each machine periodically pulls and applies
them to itself.

Repo layout:

```
inventory/
  homelab.ini          # which machines exist, in groups
playbooks/
  base.yml             # every machine: users, ssh, firewall, updates
  web.yml              # web servers: nginx + config
roles/
  ...
```

The loop on each machine — a systemd timer, not cron, so failures get
logged properly:

```ini
# /etc/systemd/system/ansible-pull.service
[Service]
Type=oneshot
ExecStart=/usr/bin/ansible-pull -U https://github.com/you/fleet.git \
  -i inventory/homelab.ini playbooks/base.yml
```

```ini
# /etc/systemd/system/ansible-pull.timer
[Timer]
OnCalendar=hourly
Persistent=true
```

```bash
systemctl enable --now ansible-pull.timer
```

**Check:** `journalctl -u ansible-pull` shows the last run; re-running
a converged machine reports `changed=0`.

Reconciliation: hourly, automatic. Drift gets overwritten — which is
the point, so make sure the team knows hand-edits don't survive.

Honest limits: Ansible is push-oriented by nature; `ansible-pull`
flips it to pull, but there's no live drift dashboard — you read the
logs. Secrets need Ansible Vault or SOPS (see
[Secrets and Safety](secrets-and-safety.md)).

## 2. Cron + idempotent apply script (one server)

The smallest real GitOps loop. A repo holds configs and a script that
applies them; a timer pulls and runs it.

```bash
# /usr/local/bin/sync.sh, run every 10 minutes by systemd timer
set -e
cd /srv/desired-state
git pull --quiet --ff-only
cp -r nginx/* /etc/nginx/
cp -r sshd/* /etc/ssh/sshd_config.d/
systemctl reload nginx
systemctl reload sshd
```

The key word is **idempotent**: running the script twice must be safe.
Copying files and reloading services qualifies; appending lines to a
file does not. Write every step so re-running is harmless.

Honest limits: no diffing, no reporting — you only know it worked
because nothing broke. Fine for one server you also watch; not for a
fleet.

## 3. etckeeper (track /etc in git)

[etckeeper](https://etckeeper.branchable.com/) puts `/etc` under
version control automatically, committing before and after every
package operation. Hand-edits show up as uncommitted changes.

```bash
sudo apt install etckeeper   # then it just works
cd /etc && git log --oneline | head
```

**Check:** edit a config by hand, then `cd /etc && git status` shows
exactly what you changed.

Honest limits: this is principles 1–2 (declarative-ish, versioned)
without 3–4 — nothing *applies* or *reconciles*. It's audit trail,
not GitOps proper. Still valuable: "who changed the firewall rule?"
becomes answerable. Pair it with pattern 1 or 2 for the full loop.

## 4. Dotfiles with chezmoi (personal machines)

GitOps for your own configuration. [chezmoi](https://www.chezmoi.io/)
keeps dotfiles in git and applies them per machine, with templates for
differences:

```bash
chezmoi init --apply you/dotfiles   # new machine, one command
```

Per-machine differences live in templates:

```
# .chezmoi.toml.tmpl
[vars]
  is_work = {{ eq .chezmoi.hostname "work-laptop" }}
```

Change a setting, commit, push; each machine `chezmoi update`s on its
own schedule (or a timer). New laptop setup goes from an afternoon to
minutes.

Honest limits: personal scope — no one else's machine should pull
your dotfiles repo. Secrets in dotfiles need chezmoi's template +
password-manager integration, never plaintext.

## 5. Compose-from-git timer (docker homelab)

For the docker-compose homelab: service definitions live in git; a
timer pulls and reconciles.

```bash
# /usr/local/bin/compose-sync.sh, every 5 minutes
set -e
cd /srv/apps
git pull --quiet --ff-only
docker compose up -d --remove-orphans
```

`docker compose up -d` is idempotent — already-correct services are
left alone, changed ones recreated. Image updates: add `docker compose
pull` before the up, on whatever cadence you want (nightly, not every
5 minutes).

Honest limits: no health-gated rollouts; a bad compose file takes the
service down until you revert. Keep the timer interval sane and the
repo's main branch protected.

## 6. Check-mode drift reporter (alert-only reconciliation)

Sometimes you don't want auto-heal — during migrations, or on
machines where surprises are worse than drift. Run the same Ansible
playbooks in **check mode** and report instead of applying:

```bash
ansible-playbook --check --diff playbooks/base.yml > /tmp/drift.txt
if grep -q "changed=" /tmp/drift.txt; then
  curl -d "drift detected on $(hostname)" https://ntfy.sh/your-topic
fi
```

You get GitOps principles 1–3 (declarative, versioned, pulled) with
reconciliation as *notification* rather than correction. A human
decides whether the drift is a problem or a pending change.

This pairs well with pattern 1: report on a short interval, auto-apply
on a longer one, or auto-apply everywhere except the machines you
flag.

## 7. Terraform provisions, GitOps configures (the split)

A common confusion: Terraform in git is not GitOps by itself (no
reconciliation loop — it's push-based). The pattern that works:

- **Terraform/OpenTofu** provisions the *existence* of things: VMs,
  networks, DNS records. Run from CI on merge (plan on PR, apply on
  merge to main).
- **Ansible-pull / agent** configures *what's on* them, on its own
  loop.

The repo boundary matters: infrastructure repo (Terraform) vs.
configuration repo (Ansible), or clearly separated folders. Don't mix
"create the VM" and "install nginx" in one apply — they fail
differently and roll back differently.

Honest limits: Terraform state files are sensitive and finicky —
remote state with locking (S3, Terraform Cloud, etc.), never local
state in a shared setup.

## Is your pattern GitOps? (the checklist)

Run any setup against the four principles:

| Principle | Question |
|---|---|
| Declarative | Is the desired state written down, not a runbook? |
| Versioned | Is it in git, with history and review? |
| Pulled automatically | Does software (not a human) apply it? |
| Reconciled | Does something keep checking, or at least report? |

Three out of four is still useful automation — just name it honestly.
The checklist is for evaluating, not gatekeeping.

Related: [GitOps Without Kubernetes](gitops-without-kubernetes.md),
[Tools Overview](tools-overview.md),
[Secrets and Safety](secrets-and-safety.md).
