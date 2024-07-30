# Whizard 常见问题处理方法

## 1. 启用可观测中心后，监控页面无数据，该如何排查？

首先检查配置与部署，集群应该开启多集群管理，在 host 集群上，我们因该看到如 [Whizard 部署架构及数据流图](advance/data-flow) 部署架构一样，已经完成 Whizard 组件的部署

## 2. 监控页面无数据，Prometheus日志中报“...conflict，out-of-order...”错误，数据无法写入

此问题偶然出现在KSE安装后启用分布式监控中心，原因为在安装完成后，Prometheus已经开始正常运行一段时间，此时重启并切换Prometheus为Agent模式，在`/prometheus/wal`目录下仍有历史的wal日志，remote write 此wal日志无法被whizard 正常接收，导致整体数据无法写入。

**处理办法:**
进入prometheus容器，删除 `/prometheus/wal` 目录下所有文件，并重启 Prometheus。

## 3. 可观测中心启用后，只有磁盘用量等个别监控有数据其他指标无数据或据出现较大偏差

此问题同样常出现在KSE安装后启用分布式监控中心，原因与上面原因类似，由于旧Prometheus的wal日志无法正常接收，导致record-rule查询计算时查询到垃圾数据，致使数据不准确且不可用。

如果重启后还无法存在问题，请排查对应 ruler 日志。

**处理办法:**
   在确认数据正常写入后，重启 host 集群里对应 cluster 的ruler。

```shell
kubectl delete -n kubesphere-monotoring-system pod ruler-{clusterName}-xx-xx
```

## 4. 如何重建 Prometheus/Whizard 的 PVC

Prometheus/Whizard 的 StatefulSet 都是通过 CR 进行控制的，我们无法直接直接暂停其副本，只能操作对应的 CR。因此重建 PVC 时需要操作 CR 现将其副本（replicas）置为 0 ，删除救的 PVC，然后再将副本恢复，Operator 会重新创建 PVC。

**处理办法:**
    操作对应的 CR，将 replicas 置为 0， 删除对应 PVC，然后恢复原本 replicas 数量。

```shell
# prometheus
kubectl edit prometheus -n kubesphere-monotoring-system k8s

# whizard
kubectl edit ingester -n kubesphere-monotoring-system whizard-local-auto-0
```
