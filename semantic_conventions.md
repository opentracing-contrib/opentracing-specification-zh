# 语义惯例

[OpenTracing标准](./specification.md) 描述的语言无关的数据模型，以及OpenTracing API的使用方法。在此数据模型中，包含了两个相关的概念 **Span Tag** 和 **(结构化的) Log Field**，尽管在标准中，已经明确了这些操作，但没有定义Span的tag和logging操作时，key的使用规范。

这些语义习惯通过这篇文档进行描述。这篇文档包括两个部分：一. 通过表格罗列出所有的tag和logging操作时，标准的key值。二.描述在特定的典型场景中，如何组合使用这些标准的key值，进行建模。

### 版本命名策略

修改此文件，将影响到OpenTracing标准的版本号。增加内容会增加小版本号，不向前兼容的改变（或新增大量内容）会增加大版本号。

## 标准的Span tag 和 log field

### Span tag 清单

Span的tag作用于 **整个Span**，也就是说，它会覆盖Span的整个事件周期，所以无需指定特别的时间戳。对于由时间点特性的事件，最好使用Span的log操作进行记录。（在本文档的下一章中进行描述）。

| Span tag 名称 | 类型 | 描述与实例 |
|:--------------|:-----|:-------------------|
| `component` | string  | 生成此Span所相关的软件包，框架，类库或模块。如 `"grpc"`, `"django"`, `"JDBI"`. |
| `db.instance` | string | 数据库实例名称。以Java为例，如果 jdbc.url=`"jdbc:mysql://127.0.0.1:3306/customers"`，实例名为 `"customers"`. |
| `db.statement` | string | 一个针对给定数据库类型的数据库访问语句。例如， 针对数据库类型 `db.type="sql"`，语句可能是 `"SELECT * FROM wuser_table"`; 针对数据库类型为 `db.type="redis"`，语句可能是 `"SET mykey 'WuValue'"`. |
| `db.type` | string | 数据库类型。对于任何支持SQL的数据库，取值为 `"sql"`. 否则，使用小写的数据类型名称，如 `"cassandra"`, `"hbase"`, or `"redis"`. |
| `db.user` | string | 访问数据库的用户名。如 `"readonly_user"` 或 `"reporting_user"` |
| `error` | bool | 设置为`true`，说明整个Span失败。译者注：Span内发生异常不等于error=true，这里由被监控的应用系统决定 |
| `http.method` | string | Span相关的HTTP请求方法。例如 `"GET"`, `"POST"` |
| `http.status_code` | integer | Span相关的HTTP返回码。例如 200, 503, 404 |
| `http.url` | string | 被处理的trace片段锁对应的请求URL。 例如 `"https://domain.net/path/to?resource=here"` |
| `message_bus.destination` | string | 消息投递或交换的地址。例如，在Kafka中，在生产者或消费者两端，可以使用此tag来存储`"topic name"`。|
| `peer.address` | string | 远程地址。 适合在网络调用的客户端使用。存储的内容可能是`"ip:port"`， `"hostname"`，域名，甚至是一个JDBC的连接串，如 `"mysql://prod-db:3306"` |
| `peer.hostname` | string | 远端主机名。例如 `"opentracing.io"`, `"internal.dns.name"` |
| `peer.ipv4` | string | 远端 IPv4 地址，使用 `.` 分隔。例如 `"127.0.0.1"` |
| `peer.ipv6` | string | 远程 IPv6 地址，使用冒号分隔的元祖，每个元素为4位16进制数。例如 `"2001:0db8:85a3:0000:0000:8a2e:0370:7334"` |
| `peer.port` | integer | 远程端口。如 `80` |
| `peer.service` | string | 远程服务名（针对没有被标准化定义的`"service"`）。例如 `"elasticsearch"`, `"a_custom_microservice"`, `"memcache"` |
| `sampling.priority` | integer | 如果大于0，Tracer实现应该尽可能捕捉这个调用链。如果等于0，则表示不需要捕捉此调用链。如不存在，Tracer使用自己默认的采样机制。|
| `span.kind` | string | 基于RPC的调用角色，`"client"` 或 `"server"`. 基于消息的调用角色，`"producer"` 或 `"consumer"`|

### Log field 清单

每个Span的log操作，都具有一个特定的时间戳（这个时间戳必须在Span的开始时间和结束时间之间），并包含一个或多个 **field**。下面是标准的field。

| Span log field 名称 | 类型    | 描述和实例 |
|:--------------------|:--------|:-------------------|
| `error.kind` | string | 错误类型（仅在`event="error"`时使用）。如 `"Exception"`, `"OSError"` |
| `error.object` | object | 如果当前语言支持异常对象（如 Java, Python），则为实际的Throwable/Exception/Error对象实例本身。例如 一个 `java.lang.UnsupportedOperationException` 实例, 一个python的 `exceptions.NameError` 实例 |
| `event` | string | Span生命周期中，特定时刻的标识。例如，一个互斥锁的获取与释放，或 在[Performance.timing](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceTiming) 规范中描述的，浏览器页面加载过程中的各个事件。 还例如，Zipkin中 `"cs"`, `"sr"`, `"ss"`, 或 `"cr"`. 或者其他更抽象的 `"initialized"` 或 `"timed out"`。出现错误时，设置为 `"error"` |
| `message` | string | 简洁的，具有高可读性的一行事件描述。如 `"Could not connect to backend"`, `"Cache invalidation succeeded"` |
| `stack` | string | 针对特定平台的栈信息描述，不强制要求与错误相关。如 `"File \"example.py\", line 7, in \<module\>\ncaller()\nFile \"example.py\", line 5, in caller\ncallee()\nFile \"example.py\", line 2, in callee\nraise Exception(\"Yikes\")\n"` |

## 典型场景建模

### RPCs

使用下面tag为RPC调用建模：

- `span.kind`: `"client"` 或 `"server"`。**在Span开始时**，设置此tag是十分重要的，它可能影响内部ID的生成。
- `error`: RPC调用是否发生错误
- `peer.address`, `peer.hostname`, `peer.ipv4`, `peer.ipv6`, `peer.port`, `peer.service`: 可选tag。描述RPC的对端信息。（一般只有在无法获取到这些信息时，才不设置这些值）

### Message Bus

消息服务是一个异步调用，所以消费端的Span和生产端的Span使用 **Follows From** 关系。(查看 [Span间关系](./specification.md#references-between-spans))

使用下面tag为消息服务建模：

- `message_bus.destination`: 上表已描述
- `span.kind`: `"producer"` 或 `"consumer"`. 建议 **在span开始时** 设置此tag，它可能影响内部ID的生成。
- `peer.address`, `peer.hostname`, `peer.ipv4`, `peer.ipv6`, `peer.port`, `peer.service`: 可选tag，描述消息服务中broker的地址。（可能在内部无法获取）

### Database (client) calls

使用下面tag为数据库客户端调用建模：

- `db.type`, `db.instance`, `db.user`, 和 `db.statement`: 上表已描述
- `peer.address`, `peer.hostname`, `peer.ipv4`, `peer.ipv6`, `peer.port`, `peer.service`: 描述数据库信息的可选tag
- `span.kind`: `"client"`

### Captured errors，捕获错误

OpenTracing中，根据语言的不同，错误可以通过不同的方式来进行描述，有一些field是专门针对错误输出的，其他则不是（例如：`event` 或 `message`）

如果存在错误对象，它其中包含栈信息和错误信息，log时使用如下的field：

- event=`"error"`
- error.object=`<error object instance>`

对于其他语言（译者注：不存在上述的错误对象），或上述操作不可行时：

- event=`"error"`
- message=`"..."`
- stack=`"..."` (可选)
- error.kind=`"..."` (可选)

通过此方案，Tracer实现可以在需要时，获取所需的错误信息。
