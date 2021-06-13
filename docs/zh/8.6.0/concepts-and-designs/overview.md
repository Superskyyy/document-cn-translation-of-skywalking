# 总览

SkyWalking 是一个开源的可观测性平台，其适用于从服务和云原生基础设施收集，分析，聚合数据并提供可视化。SkyWalking 提供了一种简便的方式来清晰地观测即便是横跨多个云平台的分布式系统。SkyWalking 更是一个现代化的，专为云原生，容器化分布式系统设计的应用程序性能监控系统 APM(Application Performance Monitoring)。

## 为什么要用 SkyWalking？

在许多不同的场景下，SkyWalking 为观测和监控分布式系统提供了解决方案。首先，像传统方式那样，SkyWalking 为服务提供了自动打点的 Agent(代理)，如 Java，C#，Node.js，Go，PHP 以及 Nginx LUA(这里我们呼吁对 Python 和 C++ SDK 的贡献)。在多语言场景和持续部署的环境中，云原生基础设施正变得更加强大，但也更为复杂。SkyWalking 提供的服务网格接收器(Service mesh receiver)可以让 SkyWalking 接收来自服务网格框架(例如 Istio/ Envoy ，Linkerd)的遥测数据，以帮助用户们理解整个分布式系统。

SkyWalking 为 **服务(service)**，**服务实例(service instance)**，以及 **端点(endpoint)** 提供了可观测能力。服务(Service)，实例(Instance) 以及 端点(Endpoint) 这几个概念在如今随处可见，所以让我们先来了解一下他们在 SkyWalking 的范畴中都代表着什么:

- **服务(Service)**： 代表对请求提供相同行为的一组工作负载。在使用打点代理或 SDK 的时候，你可以自定义服务的名称。SkyWalking 还可以使用你在 Istio 等平台中定义的名称。
- **服务实例(Service Instance)**： 上述的一组工作负载中的每一个工作负载即为一个实例。如 Kubernetes 中的 `pods`一样，服务实例未必就是操作系统中的一个进程。 然而，当你在使用打点代理的时候，一个服务实例实际就是操作系统上的一个真实进程。
- **端点(Endpoint)**： 对于某服务所接收的请求路径，例如 HTTP 的 URI 路径或 gRPC 服务的类名 + 方法签名。

用户可以通过 SkyWalking 来了解服务与端点之间的拓扑结构，并且查看每一个服务/服务实例/端点的性能指标，还可以为他们设置报警规则。

除此之外，你还可以集成

1. 其他分布式追踪 - 使用 Skywalking 原生代理和 Zipkin，Jaeger，OpenCensus 的 SDK。
2. 其他度量指标系统 - 例如 Prometheus，Sleuth(Micrometer)，OpenTelemetry。

## 架构

SkyWalking 逻辑上分为四部分: 探针(Probes)、平台后端、存储、用户界面(UI)。

![img 架构图](http://skywalking。apache。org/assets/frame-v8.jpg?u=20200423)

- **探针** 将收集的数据转化成 SkyWalking 适用的格式(不同的探针支持不同的数据来源)。
- **平台后端** 支持数据聚合，数据分析以及驱动数据流从探针到用户界面的流程。其中囊括了 Trace，Metrics 和 Logs。
- **存储** 通过开放的插件化的接口存放 SkyWalking 数据。 你可以选择既有的存储系统，如 ElasticSearch，H2 或 MySQL，TiDB，InfluxDB，也可以选择自己实现一个存储系统。 当然，我们非常欢迎你贡献新的存储系统实现。
- **用户界面** 一套高度可定制化的 Web 界面，用户通过其可视化和管理 SkyWalking 数据。

## 下回分解

- SkyWalking 的 [设计目标](project-goals.md)
- 常见问题，[为什么 SkyWalking 的架构不涉及消息队列(MQ)？](。。/FAQ/why_mq_not_involved。md)
