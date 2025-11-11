# MVP.md  
## Minimum Viable Product — ArgoCD GitOps Demonstration (Kind Cluster)

---

### 1. Introduction
This stage demonstrates a **fully functional GitOps workflow** using **ArgoCD** on a local **Kind** Kubernetes cluster.  
The goal of this MVP is to prove that ArgoCD continuously tracks the Git repository  
and automatically applies changes from the `k8s/` directory into the Kubernetes environment.

---

### 2. Environment Setup
- **Cluster:** Kind (Kubernetes-in-Docker)
- **Git Repository:** [AsciiArtify](https://github.com/samtishka/AsciiArtify)
- **Namespace:** `argocd`
- **Sync Policy:** Automatic (with Prune + Self-Heal enabled)

**Cluster verification:**
```bash
kubectl get nodes
kubectl get pods -n argocd
```

**ArgoCD UI Access (background mode):**
```bash
nohup kubectl port-forward svc/argocd-server -n argocd 8080:443 > argocd.log 2>&1 &
```

Login credentials:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret   -o jsonpath="{.data.password}" | base64 --decode
```

---

### 3. ArgoCD Application Configuration

The application tracks the `k8s/` folder of the AsciiArtify repository and automatically applies any changes to the Kubernetes cluster.

**argo-app.yaml:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: asciiartify
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/samtishka/AsciiArtify
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: asciiartify
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply it:
```bash
kubectl apply -f argo-app.yaml
```

---

### 4. Demonstration of Automatic Synchronization

**Test Scenario:**
1. Initially deployed `k8s/deployment.yaml` with **4 replicas**.
2. Committed and pushed to GitHub → ArgoCD synced automatically.
3. Updated replicas count from **4 → 2** in `k8s/deployment.yaml`.
4. Pushed again — ArgoCD detected the change and transitioned:
   - **OutOfSync → Synced**
   - Cluster pods updated automatically.

**Verification:**
```bash
kubectl get pods -l app=asciiartify
kubectl describe deployment asciiartify | grep Replicas
```

Expected result:
```
Replicas: 2 desired | 2 updated | 2 available
```

---

### 5. ArgoCD Auto-Sync Demonstration (GIF)

The following GIF shows the real-time ArgoCD synchronization process  
when the replicas value is changed from 4 to 2 and pushed to GitHub.

![ArgoCD Demo](./argo-demo2.gif)


---

### 6. Conclusion
The MVP confirms that ArgoCD successfully monitors the `AsciiArtify` GitHub repository,  
detects configuration changes under the `k8s/` path, and applies them automatically to the Kubernetes cluster.  
This completes the GitOps flow: **Git → ArgoCD → Kubernetes → Synced State** ✅
