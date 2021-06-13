# 观测分析语言
OAL(Observability Analysis Language) 在流模式(streaming mode) 下负责分析流入的数据。
OAL 专攻分析服务，服务实例和端点的 metrics，因此学习和使用起来比较容易。

自 6.3 版本以来，OAL 引擎被嵌入在 OAP 服务器的运行时中，成为了 `oal-rt`(OAL 运行时)。你可以在 `/config` 文件夹中找到 OAL 脚本，修改，然后重启服务器来使其生效。
然而，OAL 脚本任是一种编译型语言，OAL 运行时会动态生成 Java 代码。

您可以在系统环境变量中设定 “SW_OAL_ENGINE_DEBUG=Y” 来查看其生成了哪些类。

## 语法
OAL 脚本应被命名为 `*.oal`
```
// 声明 metrics.
METRICS_NAME = from(SCOPE.(* | [FIELD][,FIELD ...]))
[.filter(FIELD OP [INT | STRING])]
.FUNCTION([PARAM][，PARAM ...])

// 禁用 hard code
disable(METRICS_NAME);
```

## 作用域(Scope)
主要的 **作用域** 有 `All`，`Service`，`ServiceInstance`，`Endpoint`，`ServiceRelation`，`ServiceInstanceRelation`, `EndpointRelation`。
当然也有一些归属于某一主要作用域的次级域。请查看 [作用域定义](scope-definitions.md) 来查询所有的作用域和字段定义。


## 过滤器(Filter)
使用过滤器通过指定字段名或表达式来构建字段值的筛选条件。

表达式支持使用 `and`, `or` 和 `(...)` 进行连接.
操作符支持 `==`，`!=`，`>`，`<`，`>=`，`<=`，`in [...]` ,`like %...`，`like ...%` ，`like %...%` ，`contain` 和 `not contain`，他们可以基于字段类型进行类型检测，如果类型不兼容会在编译/代码生成期间报错.

## 聚合函数(Aggregation Function)
SkyWalking OAP 内核提供了一系列默认函数，也可以自行实现额外的函数。

提供的函数
- `longAvg` - 某个作用域实体所有输入的平均值。输入字段必须是 `long` 类型。
> instance_jvm_memory_max = from(ServiceInstanceJVMMemory.max).longAvg();

上述例子中，输入是 `ServiceInstanceJVMMemory` 域的每个请求，平均值是基于字段 `max` 进行求值的。

- `doubleAvg` - 某个作用域实体的所有输入的平均值. 输入字段必须是 `double` 类型.
> instance_jvm_cpu = from(ServiceInstanceJVMCPU.usePercent).doubleAvg();

上述例子中，输入是 `ServiceInstanceJVMCPU` 域的每个请求，平均值是基于字段 `usePercent` 进行求值的。

- `percent` - 输入中匹配指定条件的百分数。
> endpoint_percent = from(Endpoint.*).percent(status == true);

上述例子中，输入代表每一个端点的请求，而条件是 `endpoint.status == true`.

- `rate` - 比率(rate) 也是以百分数计算的，指其输入中匹配指定条件的百分数。
> browser_app_error_rate = from(BrowserAppTraffic.*).rate(trafficCategory == BrowserAppTrafficCategory.FIRST_ERROR, trafficCategory == BrowserAppTrafficCategory.NORMAL);

上述例子中，输入代表每个浏览器 app 的流量请求，`分子(numerator)` 条件 是 `trafficCategory == BrowserAppTrafficCategory.FIRST_ERROR`，`分母(denominator)` 条件是 `trafficCategory == BrowserAppTrafficCategory.NORMAL`。

参数 (1) 是 `分子(numerator)` 条件。
参数 (2) 是 `分母(denominator)` 条件。

- `count` - 某个作用域实体的调用总数。
> service_calls_sum = from(Service.*).count();

上述例子统计每个服务的请求总量。

- `histogram` - 查看 [热图 Heat map WIKI](https://en.wikipedia.org/wiki/Heat_map).
> all_heatmap = from(All.latency).histogram(100, 20);

上述例子是生成所有传入请求的热图，参数(1)是时延的精度，例如在上面的例子中，113ms 和 193ms 都被归于 101-200ms 这一组。参数(2)是分组个数。在上述情况下，一共有21(参数 + 1)组数据，分别为 0-100ms，101-200ms，… 1901 - 2000ms, 2000+ms。

- `apdex` - 查看 [应用性能指数 Apdex WIKI](https://en.wikipedia.org/wiki/Apdex).
> service_apdex = from(Service.latency).apdex(name, status);

上述例子中，该属性描述的是服务的 Apdex 应用性能指数评分。
参数(1)是服务名称，其代表的阈值可以在配置文件 service-apdex-threshold.yml 中定义。
参数(2)是该请求的状态，状态(success/ failure) 反映出 Apdex 计算。

- `p99`，`p95`，`p90`，`p75`，`p50` - 查看 [百分位数 Percentile WIKI](https://en.wikipedia.org/wiki/Percentile).
> all_percentile = from(All.latency).percentile(10);

**百分位数** 是自 7.0 版本引入的第一个多值 metric。由于有多个值，可以通过 `getMultipleLinearIntValues` GraphQL 语句进行查询。
本例查看所有传入请求的 “p99”、“p95”、“p90”、“p75”、“p50”。参数是 p99 时延计算的精度，如在上述情况下，120ms 和 124ms 被认为是相同的响应时间。
在 7.0.0 之前，'p99'、'p95'、'p90'、'p75'、'p50'函数被用来分别计算 metrics。这在 7.x 版本仍然支持，但我们不推荐使用，且并没有包括在官方 OAL 脚本中。

> all_p99 = from(All.latency).p99(10);

本例计算所有传入请求的 p99 数值。参数是 p99 时延计算的精度，如在上述情况下，120ms 和 124ms 被认为是相同的响应时间。

## Metrics 名称
存储实现、报警以及查询模块使用的的 metrics 名称。 SkyWalking 内核支持自动类型推断.

## 组(Group)
所有 metrics 数据会都会按 Scope.ID 和最小时间桶(min-level TimeBucket) 进行分组.
- 在 `端点(Endpoint)` 作用域中，Scope.ID 与 Endpoint ID 相同(也就是：基于服务和其端点的唯一 ID)。

## 禁用(Disable)
`Disable` 是 OAL 中的进阶语句，其仅在特定情况下使用。
一些聚合与 metrics 是在内核中 hard code(写死的)。例如 `segment` 和 `top_n_database_statement`。 这个 `disable` 语句的设计目的是将上述 hard code 禁用。
默认情况下他们均不会被禁用。


**请注意** - 所有禁用语句需要被写入 `oal/disable.oal` 脚本文件。 

## 一些例子
```
// Calculate p99 of both Endpoint1 and Endpoint2
endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1"，"Endpoint2")).summary(0.99)

// Calculate p99 of Endpoint name started with `serv`
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like "serv%").summary(0.99)

// Calculate the avg response time of each Endpoint
endpoint_avg = from(Endpoint.latency).avg()

// Calculate the p50，p75，p90，p95 and p99 of each Endpoint by 50 ms steps.
endpoint_percentile = from(Endpoint.latency).percentile(10)

// Calculate the percent of response status is true，for each service.
endpoint_success = from(Endpoint.*).filter(status == true).percent()

// Calculate the sum of response code in [404，500，503]，for each service.
endpoint_abnormal = from(Endpoint.*).filter(responseCode in [404，500，503]).count()

// Calculate the sum of request type in [RequestType.RPC，RequestType.gRPC]，for each service.
endpoint_rpc_calls_sum = from(Endpoint.*).filter(type in [RequestType.RPC，RequestType.gRPC]).count()

// Calculate the sum of endpoint name in ["/v1"，"/v2"]，for each service.
endpoint_url_sum = from(Endpoint.*).filter(name in ["/v1"，"/v2"]).count()

// Calculate the sum of calls for each service.
endpoint_calls = from(Endpoint.*).count()

// Calculate the CPM with the GET method for each service.The value is made up with `tagKey:tagValue`.
service_cpm_http_get = from(Service.*).filter(tags contain "http.method:GET").cpm()

// Calculate the CPM with the HTTP method except for the GET method for each service.The value is made up with `tagKey:tagValue`.
service_cpm_http_other = from(Service.*).filter(tags not contain "http.method:GET").cpm()

disable(segment);
disable(endpoint_relation_server_side);
disable(top_n_database_statement);
```
