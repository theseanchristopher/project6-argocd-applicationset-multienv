# Project 6 — Argo CD ApplicationSet Multi-Environment GitOps

This repository defines an **Argo CD ApplicationSet** that automatically manages
three environments for the same application Helm chart:

- **dev** — auto-sync enabled (continuous delivery)
- **pre** — manual sync (pre-production verification)
- **prod** — manual sync (controlled promotion)

## How It Works

- The ApplicationSet uses a **list generator** to define three environments:
  `dev`, `pre`, and `prod`.
- Each environment maps to:
  - A Kubernetes namespace: `project6-dev`, `project6-pre`, `project6-prod`
  - A values file under `environments/` in this repo
  - A child Argo CD Application named `project6-<env>`
- All Applications deploy the same Helm chart from the
  [`project4-helm-argo-gitops`](https://github.com/theseanchristopher/project4-helm-argo-gitops)
  repository at `charts/app`.

## Files and Directories

```text
applicationsets/
  project6-appset.yaml    # ApplicationSet definition (dev, pre, prod)

environments/
  dev/values.yaml         # Helm values for dev
  pre/values.yaml       # Helm values for pre
  prod/values.yaml        # Helm values for prod

