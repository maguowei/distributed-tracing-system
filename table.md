# 目前主流开源方案及对比

| 方案 | 项目地址 | 开发语言 | 背后公司或组织 | Python支持 | 侵入性 | OpenTracing 兼容 | 客户端支持语言 | UI丰富度 | 存储 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|  jaeger | https://github.com/jaegertracing/jaeger | Go | CNCF、Uber | 官方支持，较为完善 | 部分侵入 | 是 | Java, Go, Python, Node.js, C++ and C# | 中 | Memory, Cassandra, Elasticsearch, Kafka |
| zipkin | https://github.com/apache/incubator-zipkin | Java | Apache、Twitter | 第三方支持，一般 | 侵入性强 | 是 | Java, C#, Go, PHP, Python 等 https://zipkin.apache.org/pages/tracers_instrumentation.html | 中 | Memory, Cassandra, ElasticSearch and MySQL |
| Apache SkyWalking | https://github.com/apache/incubator-skywalking | Java | Apache | 暂无 | 侵入性很低 | 是 | Java, .NET Core, NodeJS and PHP | 较高 | H2、ElasticSearch 6、MySQL、TiDB https://github.com/apache/incubator-skywalking/blob/master/docs/en/setup/backend/backend-storage.md |
|Elastic APM | https://github.com/elastic/apm-server | Go | Elastic | 官方支持 | 侵入很低 | 不完善 | Go, Java, Node.js, Python, Ruby | 中 | Elasticsearch |
| cat | https://github.com/dianping/cat | Java | 美团 | 官方支持， 一般 | 侵入性强 | 否 | Java、C/C++、Python、Node.js、Go | 高 | HDFS |
| pinpoint | https://github.com/naver/pinpoint | Java | NAVER | 不支持 | 侵入很低 | 否 | Java, PHP | 高 | HBase |

注: 表格整理时间 2019年6月
