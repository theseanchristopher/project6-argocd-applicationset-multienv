# Environment Lifecycle (Dev → Pre → Prod)

All three environments are currently configured with **auto-sync** in Argo CD. Promotion to pre and prod happens by changing their values files in Project 4, not by switching sync modes.


## 1. Dev Environment

- Auto-sync  
- Updated directly by CI (via `values-dev.yaml`)  
- Used for testing latest builds  

## 2. Pre Environment

- Auto-sync  
- Currently also updated by CI (via `values-pre.yaml`) in this learning project  
- Represents a staging-like environment  

## 3. Prod Environment

- Auto-sync  
- Currently also updated by CI (via `values-prod.yaml`) in this learning project  
- Represents a production-like environment  

## 4. Promotion Path (Real-World Recommendation)

In a real production system, you would usually:

1. Auto-update **dev** from CI only.  
2. Promote a tested image tag to **pre** via Git PR into Project 4.  
3. After validation, promote the same tag to **prod** via another PR (often with approvals).  

