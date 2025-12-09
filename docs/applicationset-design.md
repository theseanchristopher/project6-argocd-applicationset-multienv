# ApplicationSet Design

This document details the full design of the ApplicationSet used in Project 6.

## 1. Purpose of the ApplicationSet

The ApplicationSet lets you define multiple Argo CD Applications from a single file.  
Project 6 uses a **list generator** to define multiple environments with minimal duplication.

## 2. List Generator Structure

The generator defines three environments:

```yaml
generators:
  - list:
      elements:
        - name: dev
          namespace: project6-dev
          valuesFile: values-dev.yaml
        - name: pre
          namespace: project6-pre
          valuesFile: values-pre.yaml
        - name: prod
          namespace: project6-prod
          valuesFile: values-prod.yaml
```

Each entry corresponds to one Argo CD Application.

## 3. Template

Every environment uses the same Helm chart and branch:

```yaml
template:
  metadata:
    name: project6-{{name}}
  spec:
    project: default
    source:
      repoURL: https://github.com/theseanchristopher/project4-helm-argo-gitops
      targetRevision: project4-gitops
      path: charts/app
      helm:
        valueFiles:
          - "{{valuesFile}}"
```

## 4. Sync Policy

The ApplicationSet enables continuous reconciliation:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
```

Namespaces (`project6-dev`, `project6-pre`, `project6-prod`) will be created automatically.

## 5. Environment Fan‑Out

The ApplicationSet generates:

| Application Name | Namespace        |
|------------------|------------------|
| project6-dev     | project6-dev     |
| project6-pre     | project6-pre     |
| project6-prod    | project6-prod    |

All environments use the same Helm chart but different values files.

## 6. Why This Matters

This design is production‑grade because it:

- Ensures consistency across all environments  
- Avoids YAML duplication  
- Keeps environment differences isolated in Project 4  
- Allows adding new environments with a single line in the generator  


## 7. Sync Policy Across Environments

All three environments use `syncPolicy.automated` with `selfHeal` and `prune` enabled. The difference between dev, pre, and prod is **which values file in Project 4 they consume**, not the sync policy itself.
