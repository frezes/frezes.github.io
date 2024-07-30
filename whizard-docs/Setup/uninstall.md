# Whizard 卸载

## KSE 中数据停止向 Whizard 写入

> 如果需要在KSE中停用可观测中心，恢复独立集群监控，可执行如下步骤:

```shell
kubectl edit cc -n kubesphere-system  ks-installer
```

```yaml
spec:
  monitoring:
    whizard:
      client:
        clusterName: ""
        gatewayUrl: ""
      enabled: false            #将whizard.enabled 置为false
      server:
        nodePort: 30990
status:
  alerting:
    enabledTime: 2023-03-13T17:36:43CST
    status: enabled             #将alerting.status: enabled  移除
  monitoring:
    enabledTime: 2023-05-31T11:47:43CST
    status: enabled             #将monitoring.status: enabled  移除
```

## Whizard 服务卸载

1. 将 `whizard-adapter` 副本置为0， 停止 KuberSphere `Cluster` 与 Whizard `Tenant` 自动同步；

    ```shell
    kubectl scale deployment --replicas=0  -n kubesphere-monitoring-system   whizard-adapter
    ```

2. 移除 Whizard 中所有租户, 此时根据租户自动伸缩的compactor、store、ruler 部署将自动卸载，ingester 因为数据保留周期限制将延后删除。因为我们要卸载operator，因此这里就直接手动删除

    ```shell
    kubectl delete tenant.monitoring.whizard.io --all
    kubectl delete ingester.monitoring.whizard.io -A --all
    ```

3. 执行 helm 卸载命令

    ```shell
    helm delete whizard -n kubesphere-monitoring-system
    ```

4. 移除部署的 whizard-agent-proxy

    ```shell
    kubectl delete deployment -n kubesphere-monitoring-system   whizard-agent-proxy
    ```

5. 至此，whizard 相关部署副本已基本卸载干净，如需卸载CRD，可执行

    ```shell
    kubectl delete crd compactors.monitoring.whizard.io gateways.monitoring.whizard.io queries.monitoring.whizard.io queryfrontends.monitoring.whizard.io routers.monitoring.whizard.io rulers.monitoring.whizard.io services.monitoring.whizard.io storages.monitoring.whizard.io storages.monitoring.whizard.io stores.monitoring.whizard.io tenants.monitoring.whizard.io
    ```
