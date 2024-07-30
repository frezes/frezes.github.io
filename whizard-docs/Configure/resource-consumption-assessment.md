# Prometheus（Whizard）资源消耗评估

## 指标数据量与磁盘容量估算

首先，Prometheus(Whizard) 采用定义的存储格式将样本数据保存在本地磁盘当中。

Prometheus 按照两个小时为一个时间窗口，将两小时内产生的数据存储在一个块（Block）中。每个块都是一个单独的目录，里面含该时间窗口内的所有样本数据（chunks），元数据文件（meta.json）以及索引文件（index）。当前样本数据所在的块会被直接保存在内存中，不会持久化到磁盘中。为了确保 Prometheus 发生崩溃或重启时能够恢复数据，Prometheus 启动时会通过预写日志（write-ahead-log(WAL)）重新记录，从而恢复数据。预写日志文件保存在 wal 目录中，每个文件大小为 128MB。wal 文件包括还没有被压缩的原始数据，所以比常规的块文件大得多。一般情况下，Prometheus 会保留三个 wal 文件，但如果有些高负载服务器需要保存两个小时以上的原始数据，wal 文件的数量就会大于 3 个。

而我们在计算磁盘消耗，一般是指的 Black 存储的资源，他的计算方式如下：

```yaml
磁盘大小 = 保留时间 * 每秒获取样本数 * 样本大小
```

1. 保留时间

    样本数据在存储中保存的时间。超过该时间限制的数据就会被删除。该参数是可以设置的。

    * 基于 Prometheus:

        ```shell
        # 你可以通过如下参数进行设置
        --storage.tsdb.retention=STORAGE.TSDB.RETENTION
                                    [DEPRECATED] How long to retain samples in storage. This flag has been deprecated, use "storage.tsdb.retention.time" instead. Use with server mode only.
        --storage.tsdb.retention.time=STORAGE.TSDB.RETENTION.TIME
                                    How long to retain samples in storage. When this flag is set it overrides "storage.tsdb.retention". If neither this flag nor "storage.tsdb.retention" nor "storage.tsdb.retention.size" is set, the retention time defaults to 15d. Units
                                    Supported: y, w, d, h, m, s, ms. Use with server mode only.
        --storage.tsdb.retention.size=STORAGE.TSDB.RETENTION.SIZE
                                    Maximum number of bytes that can be stored for blocks. A unit is required, supported units: B, KB, MB, GB, TB, PB, EB. Ex: "512MB". Based on powers-of-2, so 1KB is 1024B. Use with server mode only.
        ```

    * 基于 Prometheus-Operator：

        在 KSE/KS 中，设置的 retention 为 7d。

        ```yaml
        spec.retention: "How long to retain the Prometheus data. Default: “24h” if spec.retention and spec.retentionSize are empty."
        spec.retentionSize: "Maximum number of bytes used by the Prometheus data."
        ```

    * 基于 Whizard（Thanos） 本地存储模式:

        在 KSE 中，设置的默认 localTSDBRentension 为 7d。你可以通过全局配置文件 whizard-config 更新 Ingester.LocalTSDBRetension 或KSE 可观测中心页面配置中 本地数据存储时间 进行设置。

    * 基于 Whizard（Thanos） 对象存储模式:

        在配置对象存储的场景下，我们更推荐设置 localTSDBRentension 短一点，如 4h/8h, 配置 Store 配置 --min-time=-4h/-8h 进行查询；即本地数据存 4h， 进行查询时历史查询都查到 Store 组件中，最近 4h/8h 的数据查询在本地。 因为高数据量下，Ingester 因为需要将本地 block 上传至对象存储，无法对 Block 进行有效压缩，保存周期长时 Block 数量就非常多，反而降低查询性能；

2. PromQL 计算样本摄入率

   ```yaml
   # 每秒样本获取数
   rate(prometheus_tsdb_head_samples_appended_total[2h])
   ```

3. PromQL 计算每个样本占用bytes
   * 估算: Prometheus的压缩算法(dod&xor)，每个样本大小1~2bytes，保守按照2bytes计算；
   * 精算:

   ```yaml
    # 每个样本平均占用的bytes
    rate(prometheus_tsdb_compaction_chunk_size_bytes_sum[1h]) / rate(prometheus_tsdb_compaction_chunk_samples_sum[1h]) 
   ```

### Whizard 租户级别数据量计算及 ServiceMonitor 配置

> Whizard 需要手动设置下ServiceMonitor 配置采集，才能看到 rometheus_tsdb_* 等相关 Metrics。

* 计算各个租户每秒摄入率： `sum by(tenant)(rate(prometheus_tsdb_head_samples_appended_total{type="float"}[1h]))`

* 计算各个租户每个样本占用bytes: `avg by(tenant)(prometheus_tsdb_compaction_chunk_size_bytes_sum / prometheus_tsdb_compaction_chunk_samples_sum)`

* ServiceMonitor 配置:

    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      name: whizard-ingester
      namespace: kubesphere-monitoring-system
      labels:
        app.kubernetes.io/vendor: kubesphere
        app.kubernetes.io/name: whizard
        app.kubernetes.io/component: ingester
    spec:
      endpoints:
      - honorLabels: true
        interval: 1m
        port: http
        scheme: http
        scrapeTimeout: 30s
      jobLabel: app.kubernetes.io/name
      selector:
        matchLabels:
          app.kubernetes.io/name: ingester
    ```

## 指标数据量与内存消耗估算

上文也提高了，Prometheus 需要将当前 Block 的数据加载到内存中，在有大量的指标指标数据量时需要消耗内存，它的计算方式如链接所示: https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion/

我们可以通过上面给出的链接，计算出 Prometheus cardinality 和 ingestion 消耗的内存，除此之外，还有 query、relabel metrics 时的内存消耗，此部分比较弹性，可以在计算的基础上给出 1～4 Gi 的额外预算。

## 指标数据量与 CPU 消耗估算

CPU 消耗跟指标采集、规则计算、查询并发、查询范围及PromQL表达式等相关，这里没有明确推荐值，视场景进行调整配置。这里有一个基础要求，当你收到 `PrometheusMissingRuleEvaluations` 告警时，意味着 Prometheus 缺少规则计算，可能是由于规则组计算缓慢，CPU 不足导致的。此时就需要增加 CPU 配额了。

### 参考

1. https://yasongxu.gitbook.io/container-monitor/yi-.-kai-yuan-fang-an/di-2-zhang-prometheus/prometheus-use
2. https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion/
