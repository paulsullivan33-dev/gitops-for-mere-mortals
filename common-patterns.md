# Common Patterns

Recurring shapes that show up in real GitOps setups. Steal the ones
that fit; skip the rest.

## Environments: folders vs. branches vs. repos

You need dev, staging, and prod to differ (fewer replicas in dev,
different credentials in prod) while sharing most configuration.
Three common layouts:

- **Folders in one repo** (`envs/dev/`, `envs/prod/`): simplest,
  one review flow, easy to compare. Grows unwieldy past a handful of
  environments.
- **Branches per environment**: `dev` auto-deploys on merge,
  promotion is merging `dev` into `prod`. Familiar git mechanics;
  long-lived branches can diverge confusingly.
- **Separate repos per environment**: strongest isolation (different
  access controls per env), most overhead.

For small teams: folders. Promote by copying the tested change into
the next folder — explicitly, visibly, reviewably.

## Promotion: how changes move dev → prod

**Manual promotion.** A human copies or merges the change into the
prod definition after staging looks good. Slow, safe, auditable.
The default for anything that matters.

**Automated promotion.** The pipeline promotes after checks pass
(tests green in staging for N minutes, etc.). Faster, and exactly as
safe as your checks — which is the part to get right.

Either way, promotion is a *git operation* (merge, copy, PR), not
someone retyping values into a prod console.

## DRY configuration: base + overlays

Don't copy-paste the whole config per environment. Keep one **base**
with the shared definition, plus small **overlays** per environment
containing only the differences:

```
base/
  webapp.yaml        # 90% shared
envs/
  dev/overlay.yaml   # replicas: 1
  prod/overlay.yaml  # replicas: 5
```

(Kustomize does this natively for Kubernetes; Ansible roles and plain
templating do it elsewhere.) One place to fix the shared 90%.

## Drift detection and response

Drift — reality differing from the repo — is found by the
reconciliation loop. What happens next is a policy choice:

- **Auto-heal:** restore the desired state immediately. Strictest;
  can surprise during incidents when someone hand-fixes prod.
- **Alert only:** report the drift and let a human decide. Safer
  during incidents; lets drift linger if alerts are ignored.
- **Report in dashboard:** visible but passive. Only works if someone
  looks.

Many teams run auto-heal normally with a documented way to pause it
during incidents. Whatever you choose, choose deliberately.

## App-of-apps (Kubernetes)

One meta-definition lists all your applications; the agent deploys
*that*, which deploys everything else. Bootstrapping a whole cluster
becomes one repo. Powerful, and a single point of failure — treat the
meta-repo with extra care.

Next: [Troubleshooting](troubleshooting.md).
