# Kubernetes


## Steps to Install Prometheus:
--------------------------------
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
minikube service prometheus-server-ext
```

## Steps to install Grafana:
--------------------------
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana stable/grafana
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
minikube service grafana-ext
```
## To get user name and password in Grafana:

```bash
kubectl get secret --namespace default grafana -o yaml
echo "RkpwY21aTFNXRDVJN3Z4RWFFUjlibkV1SDBDbnFBendadmc0bmROZQ==" | openssl base64 -d ; echo

or 

kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
----------------------------------------------------------

Dashboards: https://grafana.com/grafana/dashboards/6417

```

```markdown
# Alternative Grafana Dashboards for Kubernetes Monitoring

This document provides a curated list of public Grafana.com dashboard IDs that serve as alternatives or complements to the classic "Kubernetes Cluster (Prometheus)" dashboard (ID 6417).

## Direct Cluster Overview Alternatives

These dashboards offer similar functionality to 6417 and can be used as drop-in replacements:

- **10695** – *Kubernetes Cluster (Prometheus)*
  - Newer revision with similar intent to 6417
  - Fresher layout and updated panel set
  - Recommended if you want a like-for-like but more up-to-date experience

- **315** – *Kubernetes cluster monitoring (via Prometheus)*
  - Older but very popular dashboard
  - Broad node/pod/resource coverage
  - Lighter weight and easier to customize
  - Good choice if you need a leaner, broadly adopted dashboard

## Complementary Dashboards

These dashboards fill specific gaps and work well alongside cluster overview dashboards:

- **13498** – *Kubernetes Pods monitoring via Prometheus*
  - Focuses deeper on per-pod metrics
  - Combine with a cluster overview dashboard for better drill-down capabilities
  - Useful when your main challenge is investigating noisy or high-churn pods

## Baseline Reference

- **6417** – *Original/Kubernetes Cluster (Prometheus)*
  - Keep as baseline if your team is already familiar with it
  - Consider adding one of the alternative dashboards for enhanced visibility

## Selection Guide

| Use Case | Recommended Dashboard |
|----------|---------------------|
| Modern, up-to-date replacement | 10695 |
| Lightweight, easily customizable | 315 |
| Deep pod-level monitoring | 13498 (alongside cluster overview) |
| Maintaining existing familiarity | 6417 (with complementary dashboards) |

## Import Instructions

To use any of these dashboards:

1. Navigate to: Grafana UI → Dashboards → New → Import
2. Enter the dashboard ID (e.g., 10695)
3. Click "Load"
4. Map to your Prometheus data source
5. Click "Import"

## Need More Specific Dashboards?

If you require additional monitoring perspectives, here are some categories I can help with:

- **kube-state-metrics** object health
- **node-exporter** full system metrics
- **Workload-specific** monitoring (deployments, statefulsets, etc.)
- **Networking** (service mesh, ingress, network policies)
- **Storage** (persistent volumes, storage classes)

Let me know which perspective you're interested in, and I'll provide relevant dashboard IDs.

## Additional Resources

- Comparison matrix of panel coverage
- Recommended panel pruning tips for performance optimization
- Data source configuration guidance

```