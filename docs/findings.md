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
- [x] SLOs in Sloth format (`slos/<service>/sloth.yaml`): availability (99.5% non-5xx) and
      latency (99% under 250ms for backend-api, under 1s for ml-api) over a rolling 28 days.
      Sloth generates the PrometheusRules: multi-window burn-rate alerts (availability: page +
      ticket, latency: ticket). OpenSLO was considered but dropped: Sloth cannot read
      `openslo/v1` (and generates no alerts from v1alpha), so it only added a duplicate file
- [x] SLO dashboards (official Sloth dashboards, patched for the `$Datasource` variable and the
      28d / `4w` period) in the Grafana folder "SLOs"
- [x] Monitoring the monitoring: Flux controllers scraped via a PodMonitor (port `http-prom`,
      no Service exists for it). Note: Flux v2.1+ removed `gotk_reconcile_condition`; object
      readiness now needs kube-state-metrics custom resource state (`gotk_resource_info`)
- [x] Requests/limits for every monitoring component, sized from observed peak usage
      (Grafana runs at ~600Mi, so it got a 1Gi limit). Memory limits only, no CPU limits

## TODO

- [ ] Persistent storage for postgres: it uses `emptyDir`, so a pod restart wipes the database
      (including the `documents` table). Use a PVC instead
- [ ] NetworkPolicies: by default all pod-to-pod traffic is allowed. Restrict postgres to
      backend-api, and backend-api / ml-api to load-generator and Prometheus
- [ ] Flux drift detection for HelmReleases (`spec.driftDetection.mode: enabled`): by default
      manual changes to Helm-managed resources are not reverted. Start with `warn`

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
