# Findings

## Progress

- [x] Bootstrap fixed (see finding 1)
- [x] Monitoring: kube-prometheus-stack (Prometheus, Alertmanager, Grafana, node-exporter,
      kube-state-metrics) via Flux HelmRelease, chart pinned to 92.1.0. Values in
      `values.yaml`; Prometheus on a 10Gi PVC with 7d / 8GB retention; Grafana admin password
      generated into a Secret, not stored in Git
- [x] NetworkPolicy: postgres only accepts connections from backend-api on 5432 (by default,
      Kubernetes allows all pod-to-pod traffic)

## Issues

### 1. Bootstrap fails with `namespaces "postgres" not found`

**Problem:** `flux bootstrap` returns before Flux has created the app namespaces, and
`kubectl wait` fails immediately on resources that don't exist yet.

**Fix:** `bootstrap.sh` first waits for the Flux `apps` Kustomization to be Ready, then for the
Deployments.
