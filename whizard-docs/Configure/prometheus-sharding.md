# 通过 Prometheus 分片机制将超大租户数据有效切分

## Prometheus 水平分片

因为超大的监控规模，导致需要被监控的 instance 非常庞大时，单个 Prometheus 压力过大，我们可以可以通过 Prometheus 的hashmod relabel action 来优化性能。通过这种办法，面对成千上万的 instance 时，一台 Prometheus 只需要监控其中的所有各种各样实例的一部分 instance。

在 modulus 里，配置了 4 为基数。每个 Prometheus 只抓取 1/4，比如上面的配置就只抓取 hashmod 后 __temp_hash 为 2 的 targets。抓取完成后，可以再通过 remote_write 对这 4 台 Prometheus Server 的数据进行聚合。

``` yaml
global:
  external_labels:
  env: prod
  scraper: 2
scrape_configs:
  - job_name: my_job
    ...
    relabel_configs:
      - source_labels: [__address__]
        modulus: 4
        target_label: __tmp_hash
        action: hashmod
      - source_labels: [__tmp_hash]
        regex: 2
        action: keep
```

## 通过水平分片数据切分多个 Whizard Tenant 内

上小节我们可以看到 Prometheus 可以通过 hashmod 的方式将一个超大规模的 instance 拆分到多个 Prometheus Server 中， 但数据查询时还需要聚合，就需要我们通过 remote_write 将数据写入到 Whizard 或其他方案中。这里我们借助 Prometheus-Operator 的 SHARD 机制，同时将数据写入不同 Whizard 租户内。



**Prometheus 配置：**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.46.0
  name: k8s
  namespace: monitoring
spec:
  alerting:
    alertmanagers:
    - apiVersion: v2
      name: alertmanager-main
      namespace: monitoring
      port: web
  enableFeatures: []
  evaluationInterval: 30s
  externalLabels: {}
  image: quay.io/prometheus/prometheus:v2.46.0
  nodeSelector:
    kubernetes.io/os: linux
  podMetadata:
    labels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
      app.kubernetes.io/version: 2.46.0
  podMonitorNamespaceSelector: {}
  podMonitorSelector: {}
  portName: web
  probeNamespaceSelector: {}
  probeSelector: {}
  remoteWrite:
  - url: http://172.31.73.196:30990/tenant-$(SHARD)/api/v1/receive           # 将分片变量 ${SHAED} 写入租户路径中，实现数据切分到多个租户中
  replicas: 1
  resources:
    requests:
      memory: 400Mi
  ruleNamespaceSelector: {}
  ruleSelector: {}
  scrapeInterval: 30s
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus-k8s
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector: {}
  shards: 4                                # 设置分片数，将采集的 Metrics 拆分为 4 个Prometheus 采集
  version: 2.46.0
status:
  availableReplicas: 4
  conditions:
  - lastTransitionTime: "2023-08-17T06:06:15Z"
    observedGeneration: 5
    status: "True"
    type: Available
  - lastTransitionTime: "2023-08-17T06:06:16Z"
    observedGeneration: 5
    status: "True"
    type: Reconciled
  paused: false
  replicas: 4
  shardStatuses:
  - availableReplicas: 1
    replicas: 1
    shardID: "0"
    unavailableReplicas: 0
    updatedReplicas: 1
  - availableReplicas: 1
    replicas: 1
    shardID: "1"
    unavailableReplicas: 0
    updatedReplicas: 1
  - availableReplicas: 1
    replicas: 1
    shardID: "2"
    unavailableReplicas: 0
    updatedReplicas: 1
  - availableReplicas: 1
    replicas: 1
    shardID: "3"
    unavailableReplicas: 0
    updatedReplicas: 1
  unavailableReplicas: 0
  updatedReplicas: 4
```

**Prometheus Pod 状态:**

```shell
# 查看 Prometheus Pod 状态
# kubectl  get  po -n monitoring  -l app.kubernetes.io/component=prometheus
NAME                       READY   STATUS    RESTARTS   AGE
prometheus-k8s-0           2/2     Running   0          48m
prometheus-k8s-shard-1-0   2/2     Running   0          48m
prometheus-k8s-shard-2-0   2/2     Running   0          48m
prometheus-k8s-shard-3-0   2/2     Running   0          48m
```

Prometheus-Operator 的 SHARD 机制，会自动将 ServerMonitor 的采集配置注入 hashmod，这是分片数为 0 的Prometheus 的配置文件，可以看到只采集 hashmod 为 0 的采集项。

```yaml
  - source_labels: [__address__]
    separator: ;
    regex: (.*)
    modulus: 4
    target_label: __tmp_hash
    replacement: $1
    action: hashmod
  - source_labels: [__tmp_hash]
    separator: ;
    regex: "0"
    replacement: $1
    action: keep
```
