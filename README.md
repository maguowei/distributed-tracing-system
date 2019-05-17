# 利用Jaeger打造云原生架构下分布式追踪系统

随着应用容器化和微服务的兴起，借由`Docker`和 `Kubernetes`等工具, 服务的快速开发和部署成为可能，构建微服务应用变得越来越简单。
但是随着大型单体应用拆分为微服务，服务之间的依赖和调用变得极为复杂，这些服务可能是不同团队开发的，可能基于不同的语言，服务之间的调用和通讯可能是基于rpc, 可能是基于restful api 还可能是通过消息队列等。如何在这样的环境下迅速debug和排查线上问题, 如何理清服务依赖调用关系，追踪各服务处理耗时和查找性能瓶颈, 对服务的容量评估变成一个棘手的事情。

## `可观察性`(Observability)

为了应对这些问题，可观察性概念被引入，目前定义的可观察性目前主要包含以下三个方面

- 日志(Logging)
- 度量(Metrics)
- 分布式追踪(Tracing)

![mirror](./imgs/CNCF-Observability.png)

Logging，Metrics 和 Tracing 有各自专注的部分， 这三者也有相互重叠的部分，

![mirror](./imgs/metrics-tracing-and-logging.png)

> Metric的特点是，它是可累加的：他们具有原子性，每个都是一个逻辑计量单元，或者一个时间段内的柱状图。 例如：队列的当前深度可以被定义为一个计量单元，在写入或读取时被更新统计； 输入HTTP请求的数量可以被定义为一个计数器，用于简单累加； 请求的执行时间可以被定义为一个柱状图，在指定时间片上更新和统计汇总。

> Logging的特点是，它描述一些离散的（不连续的）事件。 例如：应用通过一个滚动的文件输出debug或error信息，并通过日志收集系统，存储到Elasticsearch中； 审批明细信息通过Kafka，存储到数据库（BigTable）中； 又或者，特定请求的元数据信息，从服务请求中剥离出来，发送给一个异常收集服务，如NewRelic。

> Tracing的最大特点就是，它在单次请求的范围内，处理信息。 任何的数据、元数据信息都被绑定到系统中的单个事务上。 例如：一次调用远程服务的RPC执行过程；一次实际的SQL查询语句；一次HTTP请求的业务性ID。


关于三者关系的更详细信息可参考: [Metrics, tracing 和 logging 的关系](https://github.com/wu-sheng/me/blob/master/articles/metrics-tracing-and-logging.md#metrics-tracing-%E5%92%8C-logging-%E7%9A%84%E5%85%B3%E7%B3%BB)

日志是最传统的也是最必不可少的，目前日志已经有 `ELK` 这样的成熟方案, `Metrics` 方面 `prometheus` 已经基本上成为了事实上的标准。

下面我们重点介绍下 Tracing

分布式追踪系统可以帮助我们更好的理解系统的行为、辅助调试和排查性能问题

让Tracing 流行起来最早由于 Google 的这篇论文 [Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](https://ai.google/research/pubs/pub36356)

之后Twitter基于这篇论文开发了 Zipkin 并开源了这个项目。再之后业界百花齐放，诞生了一大批开源和商业 Tracing 系统。

## OpenTracing 标准

由于近年来各种链路监控产品层出不穷，当前市面上主流的工具既有像Datadog这样的一揽子商业方案，也有 AWS X-Ray 和 Google Stackdriver Trace 这样的云厂商产品，还有像Zipkin、jaeger 这样的开源产品。

云原生基金会(CNCF) 推出了 OpenTracing 标准，用来标准化Trace数据结构和格式。
OpenTracing 通过提供平台无关、厂商无关的 API，使得开发人员能够方便的添加（或更换）追踪系统的实现。比如从Zipkin替换成Jaeger/Skywalking等后端。

在OpenTracing中，主要定义以下基本概念

1、Trace(调用链): 一条Trace（调用链）可以被认为是一个由多个Span组成的有向无环图（DAG图）

2、Span(跨度)：可以被翻译为跨度，可以被理解为一次方法调用, 一个程序块的调用, 或者一次RPC/数据库访问.只要是一个具有完整时间周期的程序访问，都可以被认为是一个span

3、Logs：每个span可以进行多次Logs操作，每一次Logs操作，都需要一个带时间戳的时间名称，以及可选的任意大小的存储结构。

4、Tags：每个span可以有多个键值对（key:value）形式的Tags，Tags是没有时间戳的，支持简单的对span进行注解和补充。

- [OpenTracing官方标准-中文版](https://github.com/opentracing-contrib/opentracing-specification-zh)
  - [OpenTracing语义标准](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md)
  - [语义惯例](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/semantic_conventions.md)

- [支持 OpenTracing 的方案](https://opentracing.io/docs/supported-tracers/)

- [最佳实践](https://opentracing.io/docs/best-practices/)


下面主要比较一下当前的开源方案。

## 方案对比

| 方案 | 项目地址 | 开源 | 开发语言 | 背后公司或组织 | Python支持 | 侵入性 | OpenTracing 兼容 | 客户端支持语言 | UI丰富度 | 存储 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|  jaeger | https://github.com/jaegertracing/jaeger | 是 | Go | CNCF/Google、 Uber | 官方支持，较为完善 | 部分侵入 | 是 | Java, Go, Python, Node.js, C++ and C# | 中 | Memory, Cassandra, Elasticsearch, Kafka |
| zipkin | https://github.com/apache/incubator-zipkin | 是 | Java | Apache/Twitter | 第三方支持，一般 | 侵入性强 | 是 | Java, C#, Go, PHP, Python 等 https://zipkin.apache.org/pages/tracers_instrumentation.html | 中 | Memory, Cassandra, ElasticSearch and MySQL |
| Apache SkyWalking | https://github.com/apache/incubator-skywalking | 是 | Java | Apache | 暂无 | 侵入性很低 | 是 | Java, .NET Core, NodeJS and PHP | 较高 | H2、ElasticSearch 6、MySQL、TiDB https://github.com/apache/incubator-skywalking/blob/master/docs/en/setup/backend/backend-storage.md |
| cat | https://github.com/dianping/cat | 是 | Java | 美团 | 官方支持， 一般 | 侵入性强 | 否 | Java、C/C++、Python、Node.js、Go | 高 | HDFS |
| pinpoint | https://github.com/naver/pinpoint | 是 | Java | NAVER (一家韩国公司) | 不支持 | 侵入很低 | 否 | Java, PHP | 高 | HBase |
|Elastic APM | https://github.com/elastic/apm-server| 是 | Go | Elastic | 官方支持 | 侵入很低 | 不完善 | Go, Java, Node.js, Python, Ruby | 中 | Elasticsearch | 


###  个人简单总结
1. 如果是纯Java栈的应用，且对于定制化需求低，可以优先考虑侵入型低的 Apache SkyWalking 
2. 考虑多语言支持、定制化和高扩展，优先选用 Jaeger（Jaeger 与 zipkin 比较类似，但是相比之下Jaeger 有一定的后发优势）
3. 偏向于纯Web应用，无需定制化且已经有ELK的日志系统可以考虑低成本的接入 Elastic APM
4. cat 和 pinpoint 当前不支持 OpenTracing 标准不做优先考虑

## Jaeger 

- 背后有CNCF和Uber支持，开发活跃 [Jaeger Roadmap](https://www.jaegertracing.io/roadmap/)
- 完全兼容 OpenTracing 标准, 支持多种主流语言
    > Built with OpenTracing support from inception, Jaeger includes OpenTracing client libraries in several languages, including Java, Go, Python, Node.js, C++ and C#. It is a Cloud Native Computing Foundation member project.

    - 支持的语言: [Client libraries in different languages](https://github.com/jaegertracing/jaeger/issues/366)
- 丰富的采样率设置支持 https://www.jaegertracing.io/docs/1.12/sampling/

缺点:
有一定的侵入性
Jaeger更专注于链路追踪(tracing), 日志和指标功能比较弱

需要考虑的问题:

1. 性能损耗 
2. 应用级的透明  尽量减少业务的侵入，目标是尽量少改或者不用修改代码
3. 扩展性

采样率

分布式追踪系统本身也会造成一定的性能低损耗，如果完整记录每次请求，对于生产环境可能会有极大的性能损耗，一般需要进行采样设置。

当前支持四种采样率设置
1. 静态采样（sampler.type=const）  sampler.param=1 全采样， sampler.param=0 不采样
2. 按照概率百分比采样（sampler.type=probabilistic）  sampler.param=0.1 则随机采十分之一的样本
3. 采样速度限制（sampler.type=ratelimiting）  sampler.param=2.0  每秒采样两个traces
4. 远程获取采样率 (sampler.type=remote) 这个是默认配置， 可以通过配置从Agent获取采样率的动态设置。

自适应采样（Adaptive Sampling）也已经在开发计划中 https://www.jaegertracing.io/docs/1.12/sampling/#adaptive-sampler

### 功能

- 故障定位 可以看到请求的完整路径，相比离散的日志,更方便定位问题(由于真实线上环境会设置采样率，可以利用debug开关实现对特定请求的全采样)
- 依赖梳理 基于调用关系生成服务依赖图
- 容量规划与评估
- 性能分析  可以看到系统不同路径上的指标数据，
- 报警 SLA 可通过grafana实现

### 架构图解

![mirror](./imgs/jaeger/architecture.png)
![mirror](./imgs/jaeger/spans-traces.png)
![mirror](./imgs/jaeger/context-prop.png)

### 组件

- 客户端库  实现了OpenTarcing API. 可以手动也可以通过已经集成OpenTracing 的框架工具实现应用的分布式追踪， 像 Flask, Dropwizard, gRPC 等都已经有现成的集成工具库。

每当接受到新的请求，就会创建 span 并关联上下文信息（trace id、span id 和 baggage）。只有 id 和 baggage 会随请求向下传递，而所有组成 span 的其他信息，如操作名称、日志等，不会在同一个trace 的span间传递。通过采样得到的 span 会在后台异步发送到 Jaeger Agents 上。

需要注意的是虽然所有的tarces都会创建，但是只有少部分会被采样，采样到的trace会被标记并用于后续的处理和存储。默认情况下，Jaeger client 的采样率是 0.1% 
也就是 千分之一，并且可以从 Agent上取回采样率设置。

- Agent  是一个网络守护进程，监听通过 UDP 发送过来的 spans，并将其批量发送给 Collector。按设计 Agent 要作为基础设施被部署到所有主机节点。Agent 将 collector 和客户端之间的路由与发现机制抽象了出来。

- Collector  从 Agents 接收 traces，并通过一个pipeline对其进行处理。目前的pipeline会检验traces、建立索引、执行转换并最终进行存储。Jaeger的存储系统是一个可插入的组件，当前支持 Cassandra, Elasticsearch 和 Kafka. （测试环境可以支持纯内存存储)

- Query  从存储中检索 traces 并通过 一个漂亮的 UI 界面进行展现, 目前支持搜索、过滤， trace 对比依赖调用关系图等功能。

### 部署实践

#### agent 以 DaemonSet 模式部署

- [Kubernetes DNS 高阶指南](https://juejin.im/entry/5b84a90f51882542e60663cc)

```bash
env:
- name: JAEGER_AGENT_HOST
  valueFrom:
    fieldRef:
      fieldPath: status.hostIP
```

#### agent以sidecar模式部署
- https://medium.com/jaegertracing/deployment-strategies-for-the-jaeger-agent-1d6f91796d09
- https://github.com/jaegertracing/jaeger-kubernetes#deploying-the-agent-as-sidecar
- https://github.com/jaegertracing/jaeger-operator#auto-injection-of-jaeger-agent-sidecars
- [Auto injection of Jaeger Agent sidecars]()

#### HotROD example
- [Take OpenTracing for a HotROD ride](https://medium.com/opentracing/take-opentracing-for-a-hotrod-ride-f6e3141f7941)


#### Jaeger dependencies

##### 生成依赖调用关系图
Jager Query UI服务中的 dependencies 选项默认点开为空，需要设置 spark-dependencies 来生成依赖关系图。

 spark-dependencies 是一个Spark job 可以聚合和分析存储中的数据，生成服务间的依赖关系图，并将依赖链接信息持久化到存储中。
之后 jaeger query Dependencies页面就可以显示服务之间的依赖关系。

##### 监控和报警

当前Jaeger缺少自带的报警机制，但是由于存储可以使用es，配合Grafana就可以实现简单的报警监控。

### Jaeger Python 

 - [jaeger-client-python](https://github.com/jaegertracing/jaeger-client-python)
 - [opentracing-python](https://github.com/opentracing/opentracing-python)
 - [python-django](https://github.com/opentracing-contrib/python-django)
 - [OpenTracing API Contributions (Python)](https://github.com/opentracing-contrib?utf8=%E2%9C%93&q=&type=&language=python)
 - [uber-common/opentracing-python-instrumentation](https://github.com/uber-common/opentracing-python-instrumentation)

### Links

- [观察之道：带你走进可观察性](https://www.infoq.cn/article/observability-enhance)
- [关于可观察性的三大支柱，你应该了解这些](https://www.infoq.cn/article/N64dDo-kRPvYbK4jCeET)
- [Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](https://ai.google/research/pubs/pub36356)
- [Dapper，大规模分布式系统的跟踪系统(中文)](https://bigbully.github.io/Dapper-translation/)
- [OpenTracing Tutorials](https://github.com/yurishkuro/opentracing-tutorial) A collection of tutorials for the OpenTracing API
- [CNCF Cloud Native Landscape](https://landscape.cncf.io/)
- [观察之道：带你走进可观察性](https://www.infoq.cn/article/observability-enhance)
- [Metrics, tracing, and logging](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html)
- [Chaos Engineering 的历史、原则以及实践](https://www.infoq.cn/article/chaos-engineering-the-history-principles-and-practice)
- [Deployment strategies for the Jaeger Agent](https://medium.com/jaegertracing/deployment-strategies-for-the-jaeger-agent-1d6f91796d09)
- [APM和调用链跟踪](https://skywalking.apache.org/zh/blog/2019-03-29-introduction-of-skywalking-and-simple-practice.html)
- [Jaeger vs Apache Skywalking](https://blog.getantler.io/jaeger-vs-apache-skywalking/)
