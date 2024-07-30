# Whizard 服务本地持久卷挂载

> 注： 持久卷挂载配置亦可参考 [storage](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/storage.md)

**默认情况下，Whizard 会为 Ingester、Store、Compactor 等组件配置持久卷挂载，并使用默认的 StorageClass 动态创建。**

Kubernetes 支持多种存储卷。Prometheus Operator 与 PersistentVolumeClaims 一起工作，它支持在请求时配置底层 PersistentVolume。

本文档假设您对 PersistentVolumes、PersistentVolumeClaims 及其 [provisioning](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#provisioning) 有基本的了解。

## 使用动态动态存储卷

```yaml
apiVersion: monitoring.whizard.io/v1alpha1
kind: Ingester
metadata:
  name: whizard
spec:
  dataVolume:
    volumeClaimTemplate:
      spec:
        resources:
          requests:
            storage: 40Gi
```

## 使用静态存储卷

```yaml
apiVersion: monitoring.whizard.io/v1alpha1
kind: Ingester
metadata:
  name: whizard
spec:
  dataVolume:
    volumeClaimTemplate:
      spec:
        selector:
          matchLabels:
            app.kubernetes.io/name: my-example-ingester
        resources:
          requests:
            storage: 50Gi
```

> 注:  whizard v0.6.1 以下版本配置文件中暂不支持变量中存在 `.`
