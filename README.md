# Project 6 — Argo CD ApplicationSet Multi‑Environment GitOps

![Project 6 Architecture](docs/images/project6-architecture.svg)

This project extends the GitOps deployment model by introducing an **Argo CD ApplicationSet** that automatically generates one Argo CD Application per environment (**dev**, **pre**, **prod**) from a single declarative configuration.

Project 6 integrates:

- **Project 1** → CI builds & updates image tags (dev only)
- **Project 4** → Helm chart source + all `values-*.yaml`
- **Project 6** → Multi‑environment ApplicationSet configuration
- **Argo CD** → Pull‑based continuous deployment

This project maintains the exact documentation structure and tone used in **Project 5**.

---

## 1. Repository Structure

```
project6-argocd-applicationset-multienv/
│
├── applicationsets/
│   └── project6-appset.yaml         # Multi‑environment ApplicationSet
│
└── README.md
```

**Note:**  
This repository intentionally focuses only on the **Argo CD ApplicationSet configuration**.  
The Helm chart and all environment-specific values files (`values-dev.yaml`, `values-pre.yaml`, `values-prod.yaml`) live in **Project 4**, which is the single source of truth for chart configuration in both Project 5 and Project 6.

---

## 2. Deployment Flow (High‑Level)

1. A developer pushes code to **Project 1**, triggering CI.  
2. GitHub Actions builds the Docker image and pushes it to Amazon ECR.  
3. CI updates **Project 4** (`project4-gitops` branch), modifying `values-dev.yaml`.  
4. Argo CD detects changes in Project 4 and pulls the updated desired state.  
5. The **Project 6 ApplicationSet** generates Applications for:
   - `project6-dev` (auto-sync)
   - `project6-pre` (auto-sync)
   - `project6-prod` (auto-sync)
6. Argo CD deploys each environment using the correct values file from Project 4.

This creates a scalable model where **Project 4** drives configuration and **Project 6** drives orchestration.

---

## 3. ApplicationSet Overview

The core of this project is the ApplicationSet located at:

```
applicationsets/project6-appset.yaml
```

### Key Components

| Field | Purpose |
|-------|---------|
| `generators.list` | Defines the list of environments (dev, pre, prod) |
| `template.metadata.name` | Controls the generated Argo CD Application name |
| `template.spec.source` | Points to Project 4 Helm chart and values |
| `syncPolicy` | Enables automated sync & self‑healing |
| `syncOptions.CreateNamespace=true` | Auto‑creates namespaces |

Each generated Application deploys the Helm chart at:

```
repoURL: https://github.com/theseanchristopher/project4-helm-argo-gitops.git
targetRevision: project4-gitops
path: charts/app
```

---

## 4. Environment Definitions

### **Dev Environment**
- Namespace: `project6-dev`
- Values file: `values-dev.yaml` (Project 4)
- Receives updates automatically from CI  
- Used for continuous deployment testing  

### **Pre Environment**
- Namespace: `project6-pre`
- Values file: `values-pre.yaml`
- Staging-like environment  
- Auto-sync enabled (can be changed in future projects)  

### **Prod Environment**
- Namespace: `project6-prod`
- Values file: `values-prod.yaml`
- Production‑level environment  
- Auto-sync currently enabled but may become manual in later projects  

---

## 5. Argo CD Applications (Generated Automatically)

### `project6-dev`
- Auto-sync enabled  
- Uses values from Project 4 (`values-dev.yaml`)  
- Receives new images on every successful CI run  

### `project6-pre`
- Auto-sync  
- Deploys the same chart with staging configuration  

### `project6-prod`
- Auto-sync  
- Production environment mirroring the dev/pre structure  

---

## 6. Applying the ApplicationSet

To install Project 6’s Argo CD configuration:

```bash
kubectl apply -f applicationsets/project6-appset.yaml -n argocd
```

Verify ApplicationSet creation:

```bash
kubectl get applicationsets -n argocd
```

Verify applications generated:

```bash
kubectl get applications -n argocd
```

---

## 7. CI/CD Integration (From Project 1)

The GitHub Actions workflow in **Project 1**:

1. Builds the Docker image  
2. Pushes to Amazon ECR  
3. Updates **all three environment values files** in **Project 4**:

   - `values-dev.yaml`  
   - `values-pre.yaml`  
   - `values-prod.yaml`  

4. Because **all environments are auto-sync**, Argo CD continuously reconciles:

   - `project6-dev`  
   - `project6-pre`  
   - `project6-prod`  

   and all three environments roll out the new image tag.

> **Note (real-world caveat):**  
> In a production platform, you would almost never auto-update pre/prod values directly from CI like this.  
> You would typically:
> - auto-update **dev** from CI, and  
> - promote to **pre/prod** via a dedicated promotion workflow or pull requests.  
> For this learning project, updating all three values files keeps the focus on how ApplicationSets and auto-sync behave across multiple environments.

---

## 8. Why ApplicationSets?

ApplicationSets provide:

- **Scalable multi‑environment GitOps**
- **Single source of truth** for deployment configuration  
- **Reduced YAML duplication**  
- **Faster onboarding of new environments**  
- **Consistent structure across dev/staging/prod**  

This model is widely used by platform engineering teams supporting dozens of services.

---

## 9. Namespace Strategy

Project 6 deploys to:

```
project6-dev
project6-pre
project6-prod
```

Namespaces are created automatically by Argo CD via:

```
syncOptions:
  - CreateNamespace=true
```

---

## 10. Summary

Project 6 provides a complete **ApplicationSet‑based multi‑environment GitOps workflow**, integrating:

- **Project 1 (CI)**
- **Project 4 (Helm chart + values)**
- **Project 6 (ApplicationSet)**
- **Argo CD (continuous deployment)**

This builds on the foundation of Project 5 while extending the system to support three environments with minimal configuration duplication.

