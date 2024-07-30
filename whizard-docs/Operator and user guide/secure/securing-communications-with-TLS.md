# Whizard 如何开启 TLS 进行内部服务通信

Whizard 内部服务间通信使用 TLS 安全加固时要求：

> * whizard 版本 >= v0.7.0

相关上下游服务版本要求：

> * kse 版本 >= 3.4.1

## 1. 生成 HTTPS 证书

> 证书生成访问不做限制，生成完证书后，需要在对应空间创建 Secret，供 whizard 挂载使用。**证书生成时需配置其服务 DNS 域名**。
> 这里我们使用 `cert-manager` 生成证书,可以直接生成对应 Secret，也可以换用其他熟悉方式配置生成证书。

1. 安装cert-manager

    ```shell
      kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
    ```

2. 查看部署文件 `certificate.yaml`

    ```yaml
    apiVersion: cert-manager.io/v1
    kind: Issuer
    metadata:
      name: selfsigned
      namespace: kubesphere-monitoring-system
    spec:
      selfSigned: {}
    ---
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: whizard-tls-certs
      namespace: kubesphere-monitoring-system
    spec:
      isCA: true
      duration: 87600h #10 year
      secretName: whizard-tls-assets
      commonName: whizard-tls-assets
      ipAddresses:
      - 127.0.0.1
      dnsNames:
      - query-whizard-operated.kubesphere-monitoring-system.svc
      - query-frontend-whizard-operated.kubesphere-monitoring-system.svc
      - router-whizard-operated.kubesphere-monitoring-system.svc   
      subject:
        organizations:
        - cluster.local
        - cert-manager
      issuerRef:
        name: selfsigned
        kind: Issuer
        group: cert-manager.io
    ---
    apiVersion: cert-manager.io/v1
    kind: Issuer
    metadata:
      name: whizard-tls-assets
      namespace: kubesphere-monitoring-system
    spec:
      ca:
        secretName: whizard-tls-assets
    ```

3. 通过部署文件 `certificate.yaml` 在 `kubesphere-monitoring-system` 空间中生成对应指定的 `secret`。

  `kubectl apply -f certificate.yaml`

  ```sh
  issuer.cert-manager.io/selfsigned created
  certificate.cert-manager.io/whizard-tls-certs created
  issuer.cert-manager.io/whizard-tls-assets created
  ````

  `kubectl get secret -n kubesphere-monitoring-system`

  ```sh
  NAME                                         TYPE                 DATA   AGE
  whizard-tls-assets                           kubernetes.io/tls    3      25s
  ```

## 2.  为 Whizard 组件 HTTP 服务配置 TLS 证书

whizard 支持手动为相关应用组件配置 TLS 证书，进行安全加固，允许单独为件如 `query-frontend`(数据查询入口)配置，也支持为其他使用 http 服务的相关组件进行设置，他们分别是query-frontend、query、router，设置方法如下：

```shell
# 在 queryfrontend 的 cr 中添加 TLS 证书 配置
kubectl edit queryfrontend.monitoring.whizard.io -n kubesphere-monitoring-system whizard

# 在 query 的 cr 中添加 TLS 证书 配置
kubectl edit query.monitoring.whizard.io -n kubesphere-monitoring-system whizard

# router 的 cr 中添加 TLS 证书 配置
kubectl edit router.monitoring.whizard.io -n kubesphere-monitoring-system whizard
```

```shell
spec:
  httpServerTLSConfig:
    certSecret:
      name: whizard-tls-assets
      key: tls.crt
    keySecret:
      name: whizard-tls-assets
      key: tls.key
```

## 3.  更新 Kubesphere-config 中与 whizard关联的请求配置

`kubectl edit configmap -n kubesphere-system kubesphere-config`

```yaml
observability:
  enabled: true
  monitoring:
    endpoint: https://query-frontend-whizard-operated.kubesphere-monitoring-system.svc:10902  # 使用 https 地址进行访问 
    httpClientConfig:
      tlsConfig:
        insecureSkipVerify: true
alerting:
  endpoint: ""
  prometheusEndpoint: ""
  thanosRulerEndpoint: https://query-frontend-whizard-operated.kubesphere-monitoring-system.svc:10902  # 使用 https 地址进行访问
  thanosRulerHTTPClientConfig:
    tlsConfig:
      insecureSkipVerify: true
  thanosRuleResourceLabels: ""
```

## 4. 验证

* KSE 可观测中心概览页功能正常，使用 HTTPS 的端点链接 Whizard 服务后正常使用；
* 对相关组件配置 TLS 后，无法使用 HTTP 请求访问，必须使用 HTTPS 协议才可正常访问；

## 5. 其他

1. 由于 whizard 内部服务间开启 tls 通信为手动配置的高级功能，因此配置完成后需要对 KSE 进行配置同步，可观测中心页面方可正常使用；
2. 内部组件间开启 tls 通信后额外消耗待相关性能测试后提供；
