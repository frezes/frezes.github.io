# Whizard 默认参数配置

## 系统参数配置

> 该参数可以用过启用命令行参数传入修改，用于全局参数设置

```yaml
# 每个 Ingester 实例允许处理 Tenant 数量，即 Ingester 承载租户数据的能力，默认值为3
defaultTenantsPerIngester: 3

# 在空租户时 Ingester 回收周期； Ingester 会由于租户删除或迁移，导致 Ingester 上没有租户，此时需要回收；在使用对象存储时默认为3h， 在无对象存储情况下，此参数会和该Ingester 的 localTsdbRetention 保持一致。
defaultIngesterRetentionPeriod: 3h

# 在配置对象存储情况下每个 Compactor 实例允许处理 Tenant 数量，即 Compactor 承载租户数据的能力，默认值为10
defaultTenantsPerCompactor: 10
```

## 组件默认内置参数

```yaml
  gateway: 
    resources:
      limits:
        cpu: "2"
        memory: 4Gi
      requests:
        cpu: 50m
        memory: 50Mi

  router:
    resources:
      limits:
        cpu: "2"
        memory: 4Gi
      requests:
        cpu: 50m
        memory: 50Mi  

  queryFrontend:
    resources:
      limits:
        cpu: "2"
        memory: 4Gi
      requests:
        cpu: 50m
        memory: 50Mi  

  query:
    resources:
      limits:
        cpu: "2"
        memory: 4Gi
      requests:
        cpu: 50m
        memory: 50Mi  

  ingester:
    image: thanosio/thanos:v0.31.0
    resources:
      limits:
        cpu: "4"
        memory: 16Gi
      requests:
        cpu: 50m
        memory: 50Mi  
    dataVolume:
      persistentVolumeClaim:
        spec:
          resources:
            requests:
              storage: 20Gi

    disableTSDBCleanup: true
    localTsdbRetention: 7d  # 本地数据存储时间，在启用对象存储时建议配置为 2d，不必本地过多存储数据
    flags:
      - --tsdb.out-of-order.time-window=10m

  store:
    resources:
      limits:
        cpu: "1"
        memory: 4Gi
      requests:
        cpu: 100m
        memory: 500Mi
    dataVolume:
      persistentVolumeClaim:
        spec:
          resources:
            requests:
              storage: 20Gi
    # 默认提供了一个时间区间配置，其产生的 store 工作负载提供 `now - 36h` 之前数据的查询，其他时间的数据查询将请求 Ingester
    # 用户可根据对象存储中的数据保留期限，配置多时间分区。
    timeRanges:
    - maxTime: -36h
    flags:
      - --web.disable
      - --no-cache-index-header

  ruler:
    resources:
      limits:
        cpu: "2"
        memory: 4Gi
      requests:
        cpu: 50m
        memory: 50Mi 
    shards: 1
    evaluationInterval: 1m
    ruleSelectors:
      - matchLabels:
          role: alert-rules
    alertmanagersUrl:
      - dnssrv+http://alertmanager-operated.kubesphere-monitoring-system.svc:9093
```
