# Whizard 部署架构及数据流图

![Whizard 部署架构及数据流图](../img/whizard-data-flow-diagram.svg)

## 部署架构

* Host 集群
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) gateway-whizard: 数据网关，用于接收 member 集群的数据  
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) query-whizard: 监控数据聚合查询
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) query-frontend-whizard: 查询缓存
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) router-whizard:  数据写入路由器
  * [*Statefulset*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) ingester-whizard-local-auto-0(使用本地存储): 监控数据存储至本地PVC
  * [*Statefulset*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) ingester-whizard-remote-auto-0(使用对象存储时): 监控数据转储至S3
  * [*Statefulset*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  compactor-whizard-remote-xxxx(使用对象存储时): 压缩对象存储数据
  * [*Statefulset*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  store-whizard-remote-xxxx(使用对象存储时): 查询对象存储数据
  * [*Statefulset*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  ruler-{ClusterName}-0: 为每个集群独立创建，用于规则计算与单集群告警
  * [*Statefulset*](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  ruler-whizard-0:  处理多集群告警
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)  whizard-controller-manager:  

* Member 集群
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) whizard-agent-proxy:  用于

* Edge 集群
  * [*Deployment*](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) whizard-edge-gateway: 用于接收边缘节点的监控数据
