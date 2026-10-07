# Findings

## Progress

- [x] Bootstrap fixed (see finding 1)
- [x] Monitoring: kube-prometheus-stack (Prometheus, Alertmanager, Grafana, node-exporter,
      kube-state-metrics) via Flux HelmRelease, chart pinned to 92.1.0. Values in
      `values.yaml`; Prometheus on a 10Gi PVC with 7d / 8GB retention; Grafana admin password
      generated into a Secret, not stored in Git
- [x] ServiceMonitors for backend-api and ml-api (`/metrics` every 15s), next to each app
      (see finding 2)
- [x] Dashboards as code: Backend API and ML API (golden signals, database / inference,
      resources), each in its own Grafana folder
- [x] Logs: Loki (single binary, 5Gi, 7d retention) and Alloy shipping all pod logs to Loki;
      Loki added as a Grafana data source

## TODO

- [ ] Persistent storage for postgres: it uses `emptyDir`, so a pod restart wipes the database
      (including the `documents` table). Use a PVC instead
- [ ] NetworkPolicies: by default all pod-to-pod traffic is allowed. Restrict postgres to
      backend-api, and backend-api / ml-api to load-generator and Prometheus

## Issues

### 1. Bootstrap fails with `namespaces "postgres" not found`

**Problem:** `flux bootstrap` returns before Flux has created the app namespaces, and
`kubectl wait` fails immediately on resources that don't exist yet.

**Fix:** `bootstrap.sh` first waits for the Flux `apps` Kustomization to be Ready, then for the
Deployments.

### 2. App `endpoint` label is overwritten when scraping

**Problem:** the apps label requests with `endpoint` (`/process`, `/health`, ...), but the
Prometheus Operator also sets `endpoint` on every target (the port name, `http`). Prometheus keeps
its own value and renames the app's label to `exported_endpoint`, so queries filtering on
`endpoint` silently return wrong results.

**Fix:** `honorLabels: true` in the ServiceMonitors, so the app's labels win.
