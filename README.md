# GitOps for Mere Mortals

Plain-language guides to GitOps: the practice of keeping your systems
the way you want them by storing the *description* in git and letting
automation do the rest. Written for people who run real machines — no
CS degree required.

No jargon without an explanation. No "it's simple, just..." Every
concept starts with the *idea*, then the practice.

One note on framing: GitOps is usually taught as a Kubernetes thing.
It isn't only that. The core idea — desired state in git, software
that converges reality to match — works for servers, configs, dotfiles,
and plain scripts too. These guides keep that broader view.

## Start here

1. [What GitOps Is](what-gitops-is.md) — the core idea in plain words,
   and what it isn't.
2. [Core Principles](core-principles.md) — the four principles, with
   examples.
3. [The GitOps Loop](the-gitops-loop.md) — how a change flows from
   commit to running system.

## Putting it to work

4. [GitOps Without Kubernetes](gitops-without-kubernetes.md) — the
   broader framing: servers, configs, and scripts.
5. [Tools Overview](tools-overview.md) — what the tools do, honestly,
   without hype.
6. [Secrets and Safety](secrets-and-safety.md) — what doesn't go in
   git, and how to handle it.
7. [Common Patterns](common-patterns.md) — environments, promotion,
   and drift detection.

## Reference

8. [Troubleshooting](troubleshooting.md) — drift, failed syncs, and
   "it worked on my machine."
9. [Glossary](glossary.md) — quick definitions, A to Z.

## Writing criteria

Every guide follows the same [writing criteria](criteria.md): plain
words, no hype, honest limits, and checkable steps.
