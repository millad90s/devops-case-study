# Findings

## 1. Bootstrap fails with `namespaces "postgres" not found`

**Problem:** `flux bootstrap` returns before Flux has created the app namespaces, and
`kubectl wait` fails immediately on resources that don't exist yet.

**Fix:** `bootstrap.sh` first waits for the Flux `apps` Kustomization to be Ready, then for the
Deployments.
