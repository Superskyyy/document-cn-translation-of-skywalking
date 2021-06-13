# 指标分析语言(Meter Analysis Language, MAL)

Meter 系统提供了一种名为 MAL (Meter Analysis Language) 的函数式分析语言，其允许用户分析和聚合 OAP 流式系统中的 meter 数据。表达式得出的结果会被传入 agent analyzer 或 OC/Prometheus analyzer。

## 语言数据类型

在 MAL中，一个表达式或副表达式会被解析成下列两种类型其一：
 - **样本家族(Sample family)**：  一系列样本(metrics)，其包含的metrics 的名称是相同的。
 - **标量(Scalar)**： 一个简单的数值，支持 integer/long 和 floating/double。
 
## 样本家族(Sample family)

一系列样本，其作为 MAL 中的基本单位。举个例子：

```
instance_trace_count
```

上述的样本家族可能包含了下列来自外界模组(比如 agent analyzer) 的样本;

```
instance_trace_count{region="us-west",az="az-1"} 100
instance_trace_count{region="us-east",az="az-3"} 20
instance_trace_count{region="asia-north",az="az-1"} 33
```

### Tag 过滤器

MAL 支持四种操作来根据 tag 筛选一个样本家族中的样本：

 - tagEqual： 筛选与提供的字符串完全相同的 tags。
 - tagNotEqual： 筛选与提供的字符串不相同的 tags。
 - tagMatch： 筛选与提供的字符串正则匹配的 tags。
 - tagNotMatch： 筛选不与提供的字符串正则匹配的 tags。

举个例子, 这就过滤出了所有 instance_trace_count 样本中 us-west 和 asia-north 区域且 az 为 az-1 的样本。
this filters all instance_trace_count samples for us-west and asia-north region and az-1 az:

```
instance_trace_count.tagMatch("region", "us-west|asia-north").tagEqual("az", "az-1")
```
### Value 过滤器

MAL 支持六种操作来根据 value 筛选一个样本家族中的样本：
- valueEqual: 筛选与提供的值完全相同的 value。
- valueNotEqual: 筛选与提供的值不相同的 value。
- valueGreater: 筛选大于提供的值的 value。
- valueGreaterEqual: 筛选大于等于提供的值的 value。
- valueLess: 筛选小于提供的值的 value。
- valueLessEqual: 筛选小于等于提供的值的 value。

举个例子, 这就过滤出了所有 instance_trace_count 样本中 value >= 33 的样本;


```
instance_trace_count.valueGreaterEqual(33)
```
### Tag 操纵器(manipulator)
MAL 允许 tag 操纵器改变(如 增/删/更新) tags 和他们的值。

#### K8s
MAL 支持使用 K8s 元数据来操纵 tags 和他们的值。这个特性需要授权 OAP 服务器访问 K8s 的 `API Server`。

##### retagByK8sMeta
`retagByK8sMeta(newLabelName, K8sRetagType, existingLabelName, namespaceLabelName)`。 根据现有 label 的值为样本家族添加一个新的 tag。提供了一系列内部转换类型，其中包括
- K8sRetagType.Pod2Service  

添加一个 tag 到样本，使用 `service` 为键，`$serviceName.$namespace` 为值，根据 tag 键的给定值代表了 pod 的名称。
Add a tag to the sample using `service` as the key, `$serviceName.$namespace` as the value, and according to the given value of the tag key, which represents the name of a pod.

举个例子:
```
container_cpu_usage_seconds_total{namespace=default, container=my-nginx, cpu=total, pod=my-nginx-5dc4865748-mbczh} 2
```
表达式：
```
container_cpu_usage_seconds_total.retagByK8sMeta('service' , K8sRetagType.Pod2Service , 'pod' , 'namespace')
```
输出：
```
container_cpu_usage_seconds_total{namespace=default, container=my-nginx, cpu=total, pod=my-nginx-5dc4865748-mbczh, service='nginx-service.default'} 2
```

### 二元操作符

The following binary arithmetic operators are available in MAL:

 - \+ (addition)
 - \- (subtraction)
 - \* (multiplication)
 - / (division)

Binary operators are defined between scalar/scalar, sampleFamily/scalar and sampleFamily/sampleFamily value pairs.

Between two scalars: they evaluate to another scalar that is the result of the operator being applied to both scalar operands:

```
1 + 2
```

Between a sample family and a scalar, the operator is applied to the value of every sample in the sample family. For example:

```
instance_trace_count + 2
``` 

or 

```
2 + instance_trace_count
``` 

results in

```
instance_trace_count{region="us-west",az="az-1"} 102 // 100 + 2
instance_trace_count{region="us-east",az="az-3"} 22 // 20 + 2
instance_trace_count{region="asia-north",az="az-1"} 35 // 33 + 2
```

Between two sample families, a binary operator is applied to each sample in the sample family on the left and 
its matching sample in the sample family on the right. A new sample family with empty name will be generated.
Only the matched tags will be reserved. Samples with no matching samples in the sample family on the right will not be found in the result.

Another sample family `instance_trace_analysis_error_count` is 

```
instance_trace_analysis_error_count{region="us-west",az="az-1"} 20
instance_trace_analysis_error_count{region="asia-north",az="az-1"} 11 
```

Example expression:

```
instance_trace_analysis_error_count / instance_trace_count
```

This returns a resulting sample family containing the error rate of trace analysis. Samples with region us-west and az az-3 
have no match and will not show up in the result:

```
{region="us-west",az="az-1"} 0.8  // 20 / 100
{region="asia-north",az="az-1"} 0.3333  // 11 / 33
```

### Aggregation Operation

Sample family supports the following aggregation operations that can be used to aggregate the samples of a single sample family,
resulting in a new sample family having fewer samples (sometimes having just a single sample) with aggregated values:

 - sum (calculate sum over dimensions)
 - min (select minimum over dimensions)
 - max (select maximum over dimensions)
 - avg (calculate the average over dimensions)
 
These operations can be used to aggregate overall label dimensions or preserve distinct dimensions by inputting `by` parameter. 

```
<aggr-op>(by: <tag1, tag2, ...>)
```

Example expression:

```
instance_trace_count.sum(by: ['az'])
```

will output the following result:

```
instance_trace_count{az="az-1"} 133 // 100 + 33
instance_trace_count{az="az-3"} 20
```

### Function

`Duraton` is a textual representation of a time range. The formats accepted are based on the ISO-8601 duration format {@code PnDTnHnMn.nS}
 where a day is regarded as exactly 24 hours.

Examples:
 - "PT20.345S" -- parses as "20.345 seconds"
 - "PT15M"     -- parses as "15 minutes" (where a minute is 60 seconds)
 - "PT10H"     -- parses as "10 hours" (where an hour is 3600 seconds)
 - "P2D"       -- parses as "2 days" (where a day is 24 hours or 86400 seconds)
 - "P2DT3H4M"  -- parses as "2 days, 3 hours and 4 minutes"
 - "P-6H3M"    -- parses as "-6 hours and +3 minutes"
 - "-P6H3M"    -- parses as "-6 hours and -3 minutes"
 - "-P-6H+3M"  -- parses as "+6 hours and -3 minutes"

#### increase
`increase(Duration)`: Calculates the increase in the time range.

#### rate
`rate(Duration)`: Calculates the per-second average rate of increase in the time range.

#### irate
`irate()`: Calculates the per-second instant rate of increase in the time range.

#### tag
`tag({allTags -> })`: Updates tags of samples. User can add, drop, rename and update tags.

#### histogram
`histogram(le: '<the tag name of le>')`: Transforms less-based histogram buckets to meter system histogram buckets. 
`le` parameter represents the tag name of the bucket. 

#### histogram_percentile
`histogram_percentile([<p scalar>])`. Represents the meter-system to calculate the p-percentile (0 ≤ p ≤ 100) from the buckets. 

#### time
`time()`: Returns the number of seconds since January 1, 1970 UTC.


## Down Sampling Operation
MAL should instruct meter-system on how to downsample for metrics. It doesn't only refer to aggregate raw samples to 
`minute` level, but also expresses data from `minute` in higher levels, such as `hour` and `day`. 

Down sampling function is called `downsampling` in MAL, and it accepts the following types:

 - AVG
 - SUM
 - LATEST
 - MIN (TODO)
 - MAX (TODO)
 - MEAN (TODO)
 - COUNT (TODO)

The default type is `AVG`.

If users want to get the latest time from `last_server_state_sync_time_in_seconds`:

```
last_server_state_sync_time_in_seconds.tagEqual('production', 'catalog').downsampling(LATEST)
```

## Metric level function

There are three levels in metric: service, instance and endpoint. They extract level relevant labels from metric labels, then informs the meter-system the level to which this metric belongs.

 - `servcie([svc_label1, svc_label2...])` extracts service level labels from the array argument.
 - `instance([svc_label1, svc_label2...], [ins_label1, ins_label2...])` extracts service level labels from the first array argument, 
                                                                        extracts instance level labels from the second array argument.
 - `endpoint([svc_label1, svc_label2...], [ep_label1, ep_label2...])` extracts service level labels from the first array argument, 
                                                                      extracts endpoint level labels from the second array argument.

## More Examples

Please refer to [OAP Self-Observability](../../../oap-server/server-bootstrap/src/main/resources/fetcher-prom-rules/self.yaml)
