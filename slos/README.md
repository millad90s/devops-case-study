# SLOs

| Path | Purpose |
| --- | --- |
| `<service>/sloth.yaml` | The service's SLOs (source of truth) |
| `../infrastructure/configs/slos/<service>/rules.yaml` | Generated `PrometheusRule`, applied by Flux. Do not edit by hand |

| Service | SLO | Target | Window |
| --- | --- | --- | --- |
| backend-api | `/process` non-5xx | 99.5% | 28d |
| backend-api | `/process` under 250ms | 99% | 28d |
| ml-api | `/predict` non-5xx | 99.5% | 28d |
| ml-api | `/predict` under 1s | 99% | 28d |

Alerts: availability pages on fast burn and opens a ticket on slow burn; latency only tickets.

## Workflow

```bash
for s in backend-api ml-api; do
  sloth validate -i slos/$s/sloth.yaml
  sloth generate --default-slo-period=28d -i slos/$s/sloth.yaml -o infrastructure/configs/slos/$s/rules.yaml
done
```

Commit `sloth.yaml` and the generated `rules.yaml` together.
