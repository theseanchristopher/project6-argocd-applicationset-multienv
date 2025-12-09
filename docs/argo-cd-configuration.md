# Argo CD Configuration

This document explains the Argo CD‑related components used in Project 6.

## 1. Argo CD as the Deployment Engine

Argo CD continuously pulls Git state from:

- Project 4 (`project4-gitops` branch)
- Project 6 (ApplicationSet definition)

Argo CD never pushes—everything is Git‑driven.

## 2. Argo CD Applications

The ApplicationSet in Project 6 generates:

- `project6-dev`
- `project6-pre`
- `project6-prod`

Each includes:

- Source: Project 4 Helm chart  
- Destination: Corresponding namespace  
- Sync policy: automated  

## 3. Namespace Strategy

Namespaces are created automatically via:

```yaml
syncOptions:
  - CreateNamespace=true
```

This ensures:
- No manual namespace provisioning  
- Git is the only source of truth  

## 4. Auto‑Sync Behavior

Auto‑sync means:

- When Project 4 updates → Argo CD updates environment  
- Drift is corrected automatically  
- Failed syncs are reported in UI  


## 5. Sync Policy Summary

All three Project 6 Applications (`project6-dev`, `project6-pre`, `project6-prod`) use **automated sync**. Argo CD continuously reconciles them against Git. Environment promotion is controlled by changes to values files in Project 4.
