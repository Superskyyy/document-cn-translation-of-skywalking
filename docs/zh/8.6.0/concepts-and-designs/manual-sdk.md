# 手动打点 SDK

出色的 SkyWalking 社区已经贡献了下列手动打点 SDK：
- [Go2Sky](https://github.com/SkyAPM/go2sky) - 遵循 SkyWalking 数据格式的 Go SDK。
- [C++](https://github.com/SkyAPM/cpp2sky) - 遵循 SkyWalking 数据格式的 C++ SDK。

## SkyWalking 的数据格式和传播协议是什么？

请从[协议文档](../protocols/README.md) 中了解这些协议的详情.

## Envoy 追踪器(tracer)
Envoy 已经在其内部实现了 SkyWalking 追踪器。请移步至 [SkyWalking Tracer doc](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/skywalking.proto.html?highlight=skywalking) 与 [SkyWalking tracing sandbox](https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/skywalking_tracing.html?highlight=skywalking) 来获得更多信息。
