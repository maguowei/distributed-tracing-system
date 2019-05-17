# 分布式追踪系统

## 原理论文

- [Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](https://ai.google/research/pubs/pub36356)
    - [Dapper，大规模分布式系统的跟踪系统(中文)](https://bigbully.github.io/Dapper-translation/)

## OpenTracing 标准

- [OpenTracing官方标准-中文版](https://github.com/opentracing-contrib/opentracing-specification-zh)
  - [OpenTracing语义标准](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md)
  - [语义惯例](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/semantic_conventions.md)

- [支持 OpenTracing 的方案](https://opentracing.io/docs/supported-tracers/)

- [最佳实践](https://opentracing.io/docs/best-practices/)

## 方案对比

| 方案 | 项目地址 | 开源 | 开发语言 | 背后公司或组织 | Python支持 | 侵入性 | OpenTracing 兼容 | 客户端支持语言 | UI丰富度 | 存储 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|  jaeger | https://github.com/jaegertracing/jaeger | 是 | Go | CNCF/Google、 Uber | 官方支持，较为完善 | 部分侵入 | 是 | Java, Go, Python, Node.js, C++ and C# | 中 | Memory, Cassandra, Elasticsearch, Kafka |
| zipkin | https://github.com/openzipkin/zipkin | 是 | Java | Twitter | 第三方支持，一般 | 侵入性强 | 是 | Java, C#, Go, PHP, Python 等 https://zipkin.io/pages/tracers_instrumentation | 中 | Memory, Cassandra, ElasticSearch and MySQL |
| Apache SkyWalking | https://github.com/apache/incubator-skywalking | 是 | Java | Apache | 暂无 | 侵入性很低 | 是 | Java, .NET Core, NodeJS and PHP | 较高 | H2、ElasticSearch 6、MySQL、TiDB https://github.com/apache/incubator-skywalking/blob/master/docs/en/setup/backend/backend-storage.md |
| cat | https://github.com/dianping/cat | 是 | Java | 美团 | 官方支持， 一般 | 侵入性强 | 否 | Java、C/C++、Python、Node.js、Go | 高 | HDFS |
| pinpoint | https://github.com/naver/pinpoint | 是 | Java | NAVER (一家韩国公司) | 不支持 | 侵入很低 | 否 | Java, PHP | 高 | HBase |
|Elastic APM | https://github.com/elastic/apm-server| 是 | Go | Elastic | 官方支持 | 侵入很低 | 不完善 | Go, Java, Node.js, Python, Ruby | 中 | Elasticsearch | 

## Jaeger 

- 背后有CNCF和Uber支持，开发活跃 [Jaeger Roadmap](https://www.jaegertracing.io/roadmap/)
- 完全兼容 OpenTracing 标准, 支持多种主流语言
    > Built with OpenTracing support from inception, Jaeger includes OpenTracing client libraries in several languages, including Java, Go, Python, Node.js, C++ and C#. It is a Cloud Native Computing Foundation member project.

    - 支持的语言: [Client libraries in different languages](https://github.com/jaegertracing/jaeger/issues/366)
- 丰富的采样率设置支持 https://www.jaegertracing.io/docs/1.10/sampling/

### 原理图解

![mirror](./imgs/architecture.png)
![mirror](./imgs/spans-traces.png)
![mirror](./imgs/context-prop.png)

### Jaeger Python 

 - [jaeger-client-python](https://github.com/jaegertracing/jaeger-client-python)
 - [opentracing-python](https://github.com/opentracing/opentracing-python)
 - [python-django](https://github.com/opentracing-contrib/python-django)
 - [OpenTracing API Contributions (Python)](https://github.com/opentracing-contrib?utf8=%E2%9C%93&q=&type=&language=python)
 - [uber-common/opentracing-python-instrumentation](https://github.com/uber-common/opentracing-python-instrumentation)

### Links

- [OpenTracing Tutorials](https://github.com/yurishkuro/opentracing-tutorial) A collection of tutorials for the OpenTracing API