# Whizard Gateway authentication and authorization

> 本文档介绍如何为 Whizard Gateway 配置 HTTPS 访问及 Basic Auth 认证，配置仅适用于 Whizard 0.9.0 及以上版本。

## 1. 为 Gateway 配置 HTTPS 访问及 Basic Auth 认证

```yaml
apiVersion: monitoring.whizard.io/v1alpha1
kind: Gateway
metadata:
  labels:
    monitoring.whizard.io/service: kubesphere-monitoring-system.whizard
  name: whizard
  namespace: kubesphere-monitoring-system
spec:
spec:
  debug: true
  enabledTenantsAdmission: true
  image: kubesphere/whizard-monitoring-gateway:latest
  logFormat: logfmt
  logLevel: info
  nodePort: 30990
  webConfig:
    httpServerTLSConfig:     # 配置 TLS 证书, 证书生成创建参考 2.1
      certSecret:
        name: whizard-tls-assets
        key: tls.crt
      keySecret:
        name: whizard-tls-assets
        key: tls.key
    basicAuthUsers:                # 配置 Basic Auth 认证，用户名密码存储在 Secret 中，创建参考 2.2
    - username: 
        name: whizard-basicauth-secret
        key: username
      password: 
        name: whizard-basicauth-secret
        key: password
```

### 1.1 生成 TLS 证书

> 证书生成访问不做限制，生成完证书后，需要在对应空间创建 Secret，供 whizard 挂载使用。**证书生成时需配置其服务 DNS 域名**。
> 这里我们使用 `cert-manager` 生成证书,可以直接生成对应 Secret，也可以换用其他熟悉方式配置生成证书。

可参考 [使用 cert-manager 生成证书](./securing-communications-with-TLS.md)

### 1.2 创建用户登录的 Secret

创建 Secret 加密储存用户名和密码，密码生成可参考 [prometheus basic-auth](https://prometheus.io/docs/guides/basic-auth/#hashing-a-password)

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: whizard-basicauth-secret
  namespace: kubesphere-monitoring-system
data:
  password: JDJiJDEyJGhOZjJsU3N4Zm0wLmk0YS4xa1ZwU09WeUJDZklCNTFWUmpnQlV5djZrZG55VGxnV2o4MUF5   # test
  username: YWRtaW4=    # admin
type: Opaque
```

## 2. 通过 Gateway 访问 Thanos UI

请按照 [Whizard Gateway 开启 Debug 模式代理 Thanos Query UI](../faq/debug-ui.md) 配置，访问 `https://<host-ip>:30990/-/ui/` ，此时页面访问需要输入用户名密码。
这里用户名密码为 `admin` 和 `test`。

## 3. 为 Agent-Proxy 配置连接 Gateway 的认证信息

Agent-Proxy 通过 Gateway 传输数据，在 Gateway 设置服务端认证后，AgentProxy 同样需要增加客户端认证配置等，配置方式如下:

`kubectl edit deployment whizard-agent-proxy -n kubesphere-monitoring-system`, 在 `spec.template.spec.containers.args` 中增加如下`--gateway.config`:

```yaml
    spec:
      containers:
      - args:
        - --gateway.address=https://gateway-whizard-operated.kubesphere-monitoring-system.svc:9090    # Gateway 服务地址，注意为 https 地址
        - --tenant=host
        - |
          --gateway.config=                                        
              basic_auth:
                username: "admin"
                password: "test"
              tls_config:
                insecure_skip_verify: true
              transport_config:
                max_idle_conns_per_host: 100
        image: kubesphere/whizard-monitoring-agent-proxy:latest
        imagePullPolicy: IfNotPresent
        name: agent-proxy                
```
