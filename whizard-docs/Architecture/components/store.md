# Store
Store 是 Thanos 的原生组件之一，仅在使用对象存储的场景下使用。

Store-Gateway充当了Thanos Query与远程存储系统之间的中间层。它的作用是聚合来自多个远程存储系统的数据，并将其呈现为一个统一的数据源供Thanos Query使用。

具体而言，Store-Gateway具有以下功能：

1. 数据聚合：Store-Gateway能够从多个远程存储系统中获取数据块（数据的时间序列部分），然后将它们聚合在一起形成一个统一的数据视图。
    
2. 数据检索：Thanos Query可以向Store-Gateway发送查询请求，Store-Gateway会从远程存储系统中检索相应的数据块，并将它们返回给Thanos Query。
    
3. 数据展示：Store-Gateway提供了一个统一的数据源视图，使得Thanos Query可以无缝地跨多个数据源进行查询和分析。这意味着你可以在查询中跨越多个Prometheus实例、多个地理区域或不同的时间范围来分析指标数据。
    

通过Store-Gateway的使用，Thanos项目为长期存储和查询Prometheus指标数据提供了一个可扩展和可靠的解决方案。它使得用户能够在全球范围内存储、检索和分析大规模的指标数据，从而更好地了解系统性能和趋势。

更多 Store 使用说明可参考 https://thanos.io/tip/components/compact.md/