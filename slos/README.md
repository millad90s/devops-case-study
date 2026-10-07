# SLOs

| Path | Purpose |
| --- | --- |
| `common/openslo.yaml` | Shared OpenSLO objects: data source, burn-rate conditions, alert policies |
| `<service>/openslo.yaml` | The service's SLOs in OpenSLO v1 (source of truth) |
| `<service>/sloth.yaml` | Same SLOs in Sloth format (Sloth cannot read `openslo/v1`) |
| `../infrastructure/configs/slos/` | Generated `PrometheusRule`s, applied by Flux. Do not edit by hand |

## Workflow

```bash
oslo validate -f slos/common/openslo.yaml -f slos/backend-api/openslo.yaml -f slos/ml-api/openslo.yaml
for s in backend-api ml-api; do
  sloth validate -i slos/$s/sloth.yaml
  sloth generate --default-slo-period=28d -i slos/$s/sloth.yaml -o infrastructure/configs/slos/$s/rules.yaml
done
```

Commit the spec and the generated rules together.
