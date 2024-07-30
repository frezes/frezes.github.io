# Whizard-Monitoring-Gateway

`Whizard-Monitoring-Gateway` 的主要功能是接收和转发数据，并提供查询能力。

作为数据的接受网关，`Whizard-Monitoring-Gateway` 会接收并处理来自于 `Tenant` 侧上传的监控数据，并将数转储。它要求接收的数据必须是以 Prometheus remote-write 协议

1. 数据接收:  `Whizard-Monitoring-Gateway` 负责接收来自不同租户的数据，它支持以 `Prometheus remote-write` 协议进行上传，并确保数据的可靠接收。
2. 数据处理:  `Whizard-Monitoring-Gateway` 将接收到的数据进行特定处理，并转发到其他服务进行存储。
3. 数据查询:  `Whizard-Monitoring-Gateway` 具备查询能力，允许用户通过特定的接口或查询语言向其发送请求，并从存储的数据中检索所需信息。用户可以查询实时数据或历史数据，以满足监控、分析和报告的需求。