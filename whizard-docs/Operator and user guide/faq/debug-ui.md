# Whizard Gateway 开启 Debug 模式代理 Thanos Query UI

## 1. Gateway 开启 Debug 模式

修改 gateway cr，`kubectl edit gateways.monitoring.whizard.io -n kubesphere-monitoring-system whizard`，开启 debug 模式

```yaml
spec:
  debug: true                                           # 开启 Debug 模式
  enabledTenantsAdmission: true
  image: frezes/whizard-monitoring-gateway:v0.9.0
  logFormat: logfmt
  logLevel: info
  nodePort: 30990                                      # gateway nodePort 端口
```

## 2. Query 组件增加 `--web.prefix-header=X-Forwarded-Prefix` flag

修改 query cr，`kubectl edit query.monitoring.whizard.io -n kubesphere-monitoring-system whizard`，增加 `--web.prefix-header=X-Forwarded-Prefix` flag

```yaml
spec:
  envoy:
    image: envoyproxy/envoy:v1.20.2
  flags:
  - --web.prefix-header=X-Forwarded-Prefix
  image: thanosio/thanos:v0.32.1
  logFormat: logfmt
  logLevel: info
  replicaLabelNames:
  - prometheus_replica
  - receive_replica
  - ruler_replica
```

## 3. 使用 Gateway NodePort 端口访问 Thanos Query UI

配置好上述两步后，我们可以使用  `https://<host-ip>:30990/-/ui/` 直接访问 Thanos Query UI 页面。
