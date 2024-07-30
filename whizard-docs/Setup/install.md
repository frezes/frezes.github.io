# Whizard 部署

## 1. 使用 Helm 快速部署和管理 Whizard

helm 独立部署方式可以在kubernetes环境下完成独立部署，无需过多依赖，可使用 whizard 离线部署包快速完成部署。（附: [Whizard 部署包及离线镜像地址](https://cwiki.yunify.com/pages/viewpage.action?pageId=141383072)）

```shell
# 安装依赖的 prometheusrule CRD

kubectl apply -f https://github.com/prometheus-operator/kube-prometheus/blob/main/manifests/setup/0prometheusruleCustomResourceDefinition.yaml

# 获取Whizard部署包并使用helm安装

helm install whizard . -n kubesphere-monitoring-system --create-namespace
```

## 2. 在 KubeSphere Enterprise 中启用Whizard 可观测中心

### 2.1 在 KSE 3.X 版本启用 Whizard 可观测中心

> 在KubeSphere 3.3.1 版本首次集成 Whizard 可观测中心，在 3.X 版本通过 `ks-installer` 集成部署。**Whizard 可观测中心要求启用多集群管理**。

**注: 如果已经安装了 KubeSphere Enterprise，再启用Whizard可观测中心时，需要将status.monitoring.status 字段置空**
`kubectl edit cc -n kubesphere-system ks-installer`

```yaml
status:
  monitoring:
    status: ""            #将status从'enabled'置空以重新部署monitoring
  ……
```

#### 2.1.1 在host 集群启用 Whizard 可观测中心服务

当您在 Kubernetes 上安装 KSE 时并启用Whizard 可观测中心服务时，服务会通过 NodePort 方式暴露 30990 端口，供 member 集群将数据推送至 host 集群。

1. 下载 cluster-configuration.yaml 文件并进行编辑。

   ```shell
   vi cluster-configuration.yaml
   ```

2. 在 cluster-configuration.yaml 文件中，搜索 whizard 字段，将 enabled 的 false 更改为 true 以启用分布式可观测中心，完成后保存文件。

   ```yaml
   monitoring:
     whizard:
       client:              #在host集群无需编辑client配置
         clusterName: ''
         gatewayUrl: ''
       enabled: true        #将“false”更改为“true”。
       server: 
         nodePort: 30990    #默认在host集群开放30990端口与client进行通信，可根据实际网络模型自行配置
   ```

3. 执行以下命令开始安装：

   ```shell
   kubectl apply -f cluster-configuration.yaml
   ```

#### 2.1.2 在 member 集群启用 Whizard client服务，将数据接入 host 集群

1. 下载 cluster-configuration.yaml 文件并进行编辑。

   ```shell
   vi cluster-configuration.yaml
   ```

2. 在 cluster-configuration.yaml 文件中，搜索 whizard 字段，将 enabled 的 false 更改为 true,并**配置 gatewayUrl 及 clusterName** 以启用分布式可观测中心的 client 端，完成后保存文件。

   ```yaml
   monitoring:
     whizard:
       client:              
         clusterName: ''    #member集群名称，必须与kubeSphere多集群管理中注册的集群名称保持一致
         gatewayUrl: ''     #gateway服务暴露地址, 默认为http://{host_cluster_node_ip}:30990
       enabled: true        #将“false”更改为“true”。
       server:              #在member集群无需编辑server配置
         nodePort: 30990    
   ```

3. 执行以下命令开始安装：

   ``` shell
   kubectl apply -f cluster-configuration.yaml
   ```

### 2.2 在KSE 4.X 版本启用Whizard

TDB
