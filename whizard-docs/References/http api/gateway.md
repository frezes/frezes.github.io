# Gateway HTTP API

## 全局

| URI                               | Method    | description/Referencs                                                                                                           |
|:--------------------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------- |
| /api/v1/query                     | GET, POST | [instant-queries](https://prometheus.io/docs/prometheus/latest/querying/api/#instant-queries)                                   |
| /api/v1/query_range               | GET, POST | [range-queries](https://prometheus.io/docs/prometheus/latest/querying/api/#range-queries)                                       |
| /api/v1/series                    | GET       | [finding-series-by-label-matchers](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers) |
| /api/v1/labels                    | GET       | [getting-label-names](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)                           |
| /api/v1/label/\<label_name\>/values | GET       | [querying-label-values](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values)                       |
| /api/v1/rules                     | GET       | [rules](https://prometheus.io/docs/prometheus/latest/querying/api/#rules)                                                       |

## 租户级

| URI                               | Method    | description/Referencs                                                                                                           |
|:--------------------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------- |
| /{tenant_id}/api/v1/query                     | GET, POST | [instant-queries](https://prometheus.io/docs/prometheus/latest/querying/api/#instant-queries)                                   |
| /{tenant_id}/api/v1/query_range               | GET, POST | [range-queries](https://prometheus.io/docs/prometheus/latest/querying/api/#range-queries)                                       |
| /{tenant_id}/api/v1/series                    | GET       | [finding-series-by-label-matchers](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers) |
| /{tenant_id}/api/v1/labels                    | GET       | [getting-label-names](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)                           |
| /{tenant_id}/api/v1/label/\<label_name\>/values | GET       | [querying-label-values](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values)                       |
| /{tenant_id}/api/v1/rules                     | GET       | [rules](https://prometheus.io/docs/prometheus/latest/querying/api/#rules)                                                       |
| /{tenant_id}/api/v1/receive                   | Post      | [remote-write-receiver](https://prometheus.io/docs/prometheus/latest/querying/api/#remote-write-receiver)                       |

## 其他

| URI        | Method | description/Referencs |
|:---------- |:------ |:--------------------- |
| /-/ui      | GET    | proxy Thanos Query UI |
| /-/ready   | GET    |                       |
| /-/healthy | GET    |                       |
| /metrics   | GET    |                       |
