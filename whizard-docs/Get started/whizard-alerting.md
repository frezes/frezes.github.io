# Whizard 告警简介与演进计划

## 1. Whizard 对开源社区告警方案的增强

- 对告警规则更高维度、更细粒度的管理:
    从用户权限和告警规则的生效范围以规则组为单位定义了以下三个 CRD，以便于实现多集群、多租户、多层级的告警功能：
  - GlobalRuleGroup：对应的实例可以配置对所有集群或若干个指定的集群生效，平台管理员角色的用户可以进行这类规则组的管理。
  - ClusterRuleGroup: 仅对其实例所在的集群生效，集群管理员角色的用户可以为所管理的特定集群配置这类规则组进行告警。
  - RuleGroup: 仅对其实例所在的项目生效，项目下的成员可以管理该类规则组。

    所有规则组产生的告警消息除了告警目标的标签，还通过集群、项目、层级等标签来丰富和细化，以期提供更加精准定位告警源的信息。

    **Whizard 告警规则定义与 Prometheus 社区告警规则定义对比**

    |            | GlobalRuleGroup/ClusterRuleGroup/RuleGroup          | PrometheusRule  |
    | ---------- | ---------------------------- | ------------------------ |
    | 配置粒度 | 以单个规则组为配置单位，可以精细控制                     | 每个实例包含多个规则组，不易定位 |
    | 并发性 | 支持多个规则组并发更新 | 并发修改多个规则组容易冲突 |
    | 多集群 | 根据资源层级和配置自动控制生效集群 | 不支持|
    | 多租户 | 根据资源层级和位置自动注入权限过滤条件 | 不支持|

    > GlobalRuleGroup/ClusterRuleGroup/RuleGroup 还提供了与 PrometheusRule 的兼容性支持。

- 分片机制应对高并发大规模数据集告警
    通过分片机制，Whizard Ruler 可以提高告警方面的扩展性和性能，并允许处理大规模数据集；
- 跨集群告警
    Whizard Ruler 允许跨多个集群对告警规则进行集中管理和配置。这意味着您可以使用单个Ruler实例来管理和触发多个集群中的告警，而无需为每个集群配置独立的告警规则。这样可以简化告警配置和管理的工作流程；
- 高可用性
    Whizard Ruler支持高可用性配置，通过部署多个 Ruler 实例和使用负载均衡器，可以确保告警系统的高可靠性和容错能力。如果一个 Ruler 实例发生故障，其他实例可以接管其功能，从而避免单点故障；
- 数据持久化：
    Whizard Ruler 可将告警规则(Alerting Rule)和数据预处理规则(Recording Rule) 的计算结果分租户（集群）持久化到相应租户的长期存储系统中（如对象存储）。这使得告警数据的长期保存成为可能，您可以对历史告警进行回顾和分析

## 2. Whizard 与 ThanosRuler（Prometheus-Operator） 的对比

|            | WhizardRuler                 | ThanosRuler              |
| ---------- | ---------------------------- | ------------------------ |
| 跨集群告警 | 天然支持                     | 可实现，依赖手动繁琐配置 |
| 高可用性   | 健壮                         | 健壮                     |
| 分片能力   | 支持                         | 不支持                   |
| 定制程度   | 与 KubeSphere 告警有深度定制 | 依赖手动配置             |
| 灵活程度   | 当前稍弱，功能演进中           | 良好                     |
| 数据持久化 | 天然支持                     | 额外配置存储                 |

## 3. Whizard 后续针对外部数据源进行告警的功能演进

当前的设计中， Whizard 是贴合 KubeSphere 设计实现的， 在设计之初，没有过多考虑接入外部数据源, 因此当前直接接入外部数据源进行告警需要有一些是适配的工作。

1. Whizard 接入外部数据源进行告警处理

    以下提供了一个示例，可以支持 whizard ruler 查询外部数据源进行相关告警规则的评估和告警消息的下发 (该特性预计将在 whizard v0.7.0 发布和集成到 KSE v3.4.1)

    ```yaml
    apiVersion: monitoring.whizard.io/v1alpha1
    kind: Ruler
    metadata:
    name: deepflow
    namespace: kubesphere-monitoring-system
    spec:
    image: "thanosio/thanos:v0.31.0"
    replicas: 2
    shards: 1
    logLevel: info
    logFormat: logfmt
    evaluationInterval: 1m
    ruleSelectors:
        - matchLabels:
            rule: deepflow
    flags:
    - --query=http://{deepflow_query_endpoint} # 允许外部数据接入，对其进行查询告警
    alertmanagersUrl:
        - dnssrv+http://alertmanager-operated.kubesphere-monitoring-system.svc:9093 # push 告警消息到 KubeSphere 的告警通知系统
    ```

2. KubeSphere 支持为外部数据源配置告警规则

    目前通过 console 界面配置的规则组，会默认绑定到平台内部的监控数据源，然后由相关的 ruler 进行告警处理。
    后续将提供为外部数据源配置告警规则组的界面化支持，需要从设计和前后端协同上提供这种支持。

    当前仍可通过配置 PrometheusRule 的实例来绑定到上面为 deepflow 创建的 ruler 上进行告警(PrometheusRule 资源的 labels 需要能被 spec.ruleSelectors 匹配)。
