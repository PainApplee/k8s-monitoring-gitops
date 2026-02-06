# Kubernetes Monitoring GitOps (Argo CD)

This repository contains a **fully GitOps-driven Kubernetes monitoring stack** built with **Argo CD**.

After the initial bootstrap, **all resources are managed exclusively via Git**.  
Any manual `kubectl apply` (except installing Argo CD itself) is **not part of the workflow**.

---

## Stack Components

- Argo CD (GitOps controller)
- Prometheus
- Grafana
- Loki
- Promtail
- ingress-nginx

All components are deployed using **ApplicationSet + Helm charts** stored directly in this repository.

---

## GitOps Workflow

1. **Manually** install Argo CD (one-time action)
2. **Apply once** the `bootstrap.yaml`
3. From this point on:
   - Argo CD creates all ApplicationSets automatically
   - ApplicationSets generate Applications
   - All changes are performed **only through Git**
   - `auto-sync`, `prune`, and `self-heal` are enabled

---

## Installation

### 1. Install Argo CD (the only manual step in the cluster)
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Apply the bootstrap Application
```bash
kubectl apply -f argocd/bootstrap.yaml
```

After this step, no further manual applies are required.

---

## ⚠️ Required Post-Deployment Step

After the system is up and ingress-nginx has created its Service, you must manually retrieve the external IP of the ingress controller:
```bash
kubectl -n ingress-nginx get svc ingress-nginx-controller
```

Take the value from the `EXTERNAL-IP` column and update it in:

- `monitoring/charts/grafana/values.yaml`
- `monitoring/charts/prometheus/values.yaml`

Replace the `{LP_IP}` placeholder with the actual external IP.

### Example

**Before:**
```yaml
hosts:
  - grafana.{LP_IP}.nip.io
```

**After:**
```yaml
hosts:
  - grafana.203.0.113.15.nip.io
```

**⚠️ Without this step, Grafana and Prometheus will not be accessible externally.**

After committing the change:
- Argo CD will automatically sync the updates
- No manual sync is required

---

## Service Access

Once the correct EXTERNAL-IP is configured:

- **Grafana:**  
  `http://grafana.<EXTERNAL-IP>.nip.io`

- **Prometheus:**  
  `http://prometheus.<EXTERNAL-IP>.nip.io`

---

## Important Notes

- ❌ **Do not** use `kubectl apply` for monitoring resources
- ✅ **All changes must go through Git**
- 🔄 Argo CD will automatically revert any configuration drift
- 🧹 Removing resources from Git will remove them from the cluster (prune enabled)

---

## Additional Notes

- Uses the **App of Apps** pattern
- `sync-wave` annotations ensure correct deployment order
- Helm charts are stored directly in this repository
- The setup is ready for extension (additional services or clusters)
