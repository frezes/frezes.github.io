# 通过数据分区/分片机制扩展 Store

## 按时间分区扩展 Store

对象存储中的长期数据，通常具有较长的时间跨度。按时间进行分区，多个 store 工作负载并发读取不同时间范围的数据，可在降低单个 store 工作负载压力的同时，提高整体的查询效率。  

### 配置多个时间分区

在 `Store` CR 实例的 `timeRanges` 配置项中设置多个时间区间，将产生多个工作负载，它们分别提供不同时间区间的数据查询。  

以下示例配置了两个分区：`now - 30d` 之前的数据为一个分区，`now - 30d` 到 `now - 36h` 之间的数据划分为另一个分区。

```yaml
apiVersion: monitoring.whizard.io/v1alpha1
kind: Store
metadata:
  name: whizard
spec:
  ...
  timeRanges:
  - minTime: ''   # minTime 为空或未配置时，对应的 store 工作负载将使用默认值 0000-01-01T00:00:00Z
    maxTime: -30d
  - minTime: -30d
    maxTime: -36h # maxTime 为空或未配置时，对应的 store 工作负载将使用默认值 9999-12-31T23:59:59Z
  ...
```

> `maxTime` 和 `minTime` 还支持配置 RFC3339 格式时间，例如：`2018-01-01T00:00:00Z`

[默认内置参数](./configuration-parameters.md#组件默认内置参数) 中的 `store` 项提供了时间分区的默认配置，用户更新该默认配置项后，将在所有未配置 `timeRanges` 的 `Store` CR 实例中生效。