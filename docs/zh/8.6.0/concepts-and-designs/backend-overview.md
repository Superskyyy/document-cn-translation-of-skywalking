# 观测分析平台
SkyWalking 是一个观测分析平台(Observability Analysis Platform)。？？？提供了完整的观测性。

## 能力

SkyWalking 涵盖了可观测性中的全部三个方面，即 Tracing，Metrics 和 Logging。

- **Tracing** - SkyWalking 原生数据格式, Zipkin V1 和 V2 数据格式, 以及 Jaeger 数据格式.
- **Metrics** - SkyWalking 集成了 Istio, Envoy 和 Linkerd 等服务网格平台, 并在数据平面(Data Panel) 和控制平面(Control Panel) 提供可观测性。此外, SkyWalking 原生代理还可以以 metrics 模式运行, 这极大提升了性能。
- **Logging** - 包括了从磁盘或网络渠道采集的日志。原生代理可以自动将 trace 上下文与日志关联，或另 SkyWalking 借助文本内容将他们关联起来。

SkyWalking 设计并提供了三种强大的原生语言引擎，他们被用来分析上文中的三类可观测性数据。

1. [观测分析语言(Observability Analysis Language)](oal.md) 负责处理原生 tracing 信息和服务网格数据。
1. [指标分析语言(Meter Analysis Language)](mal.md) 负责为原生 meter 数据计算 metrics，而且采用了稳定且泛用的 metrics 系统，如 Prometheus 和 OpenTelemetry。
1. [日志分析语言(Log Analysis Language)](lal.md) 专注于日志内容，且与 MAL 协同工作。
