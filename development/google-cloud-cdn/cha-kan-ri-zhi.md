# 查看日志



每个 Cloud CDN 请求都会记录在 [Cloud Logging](https://cloud.google.com/logging/docs/quickstart-sdk) 中。Logging 会自动运行，因此无需启用。

Cloud CDN 的日志与您的 Cloud CDN 后端所连接的外部 HTTP\(S\) 负载平衡器相关联。Cloud CDN 日志相继按[转发规则](https://cloud.google.com/load-balancing/docs/forwarding-rule-concepts)和[网址映射](https://cloud.google.com/load-balancing/docs/url-map-concepts)被编入索引。

要查看 Cloud CDN 日志，请按照以下步骤操作。[控制台](https://cloud.google.com/cdn/docs/logging#%E6%8E%A7%E5%88%B6%E5%8F%B0)

1. 在 Google Cloud Console 中，转到**日志查看器**页面。

   [转到“日志查看器”页面](https://console.cloud.google.com/logs?service=network.googleapis.com)

2. 如果您想查看所有日志，请在行中的第一个下拉菜单中选择 **Cloud HTTP 负载平衡器 &gt; 所有转发规则**。
3. 如果您只想查看一个转发规则的日志，请从该列表中选择一个转发规则名称。
4. 如果您想查看某个转发规则使用的一个网址映射的日志，请选择 **Cloud HTTP 负载平衡器**，然后选择您感兴趣的转发规则和网址映射。

### 日志中的缓存命中示例 <a id="example_cache_hit_in_log"></a>

以下日志条目显示了缓存命中。要查看缓存命中，请按照以下步骤操作。[控制台](https://cloud.google.com/cdn/docs/logging#%E6%8E%A7%E5%88%B6%E5%8F%B0)

1. 在 Google Cloud Console 中，转到**日志查看器**页面。

   [转到“日志查看器”页面](https://console.cloud.google.com/logs?service=network.googleapis.com)

2. 按转发规则名称过滤。

   ```text
   2020-06-08 16:41:30.078 PDT
   GET
   304
   157 B
   null
   Chrome 83
   http://LOAD_BALANCER_IP_ADDRESS/static/us/three-cats.jpg

   CLIENT_IP_ADDRESS - "GET http://LOAD_BALANCER_IP_ADDRESS/static/us/three-cats.jpg" 304 157 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36"
   Expand all | Collapse all{
   httpRequest: {
   cacheHit: true
   cacheLookup: true
   remoteIp: "CLIENT_IP_ADDRESS"
   requestMethod: "GET"
   requestSize: "577"
   requestUrl: "http://LOAD_BALANCER_IP_ADDRESS/static/us/three-cats.jpg"
   responseSize: "157"
   status: 304
   userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36"
   }
   insertId: "1oek5rg3l3fxj7"
   jsonPayload: {
   @type: "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry"
   cacheId: "SFO-fbae48ad"
   statusDetails: "response_from_cache"
   }
   logName: "projects/PROJECT_ID/logs/requests"
   receiveTimestamp: "2020-06-08T23:41:30.588272510Z"
   resource: {
   labels: {
    backend_service_name: ""
    forwarding_rule_name: "FORWARDING_RULE_NAME"
    project_id: "PROJECT_ID"
    target_proxy_name: "TARGET_PROXY_NAME"
    url_map_name: "URL_MAP_NAME"
    zone: "global"
   }
   type: "http_load_balancer"
   }
   severity: "INFO"
   spanId: "7b6537d3672e08e1"
   timestamp: "2020-06-08T23:41:30.078651Z"
   trace: "projects/PROJECT_ID/traces/241d69833e64b3bf83fabac8c873d992"
   }
   ```

### 记录的内容 <a id="what_is_logged"></a>

除了大多数日志中包含的一般信息（例如严重性、项目 ID、项目编号和时间戳）之外，[HTTP\(S\) 负载平衡日志](https://cloud.google.com/load-balancing/docs/https/https-logging-monitoring#logging)还包含以下内容：

* [HttpRequest](https://cloud.google.com/logging/docs/reference/v2/rest/v2/LogEntry#HttpRequest) 日志字段，用于捕获 HTTP 状态代码、返回的字节数以及是否执行了缓存查询和/或缓存填充操作。
* `jsonPayload.cacheId` 字段，指示提供缓存响应的位置和缓存实例。例如，由阿姆斯特丹的缓存处理的一条缓存响应的 cacheId 值是 `AMS-85e2bd4b`，其中 `AMS` 是 [IATA 代码](https://en.wikipedia.org/wiki/IATA_airport_code)、`85e2bd4b` 是缓存实例的不透明标识符，因为某些 Cloud CDN 位置具有多个单独的缓存。
* `structPayload` 的 [`statusDetails`](https://cloud.google.com/load-balancing/docs/https/https-logging-monitoring#what_is_logged) 字段。

您可以对以下字段进行过滤，以确定 Cloud CDN 所传送的请求的缓存命中、未命中或重新验证状态：

* **缓存命中**

  `statusDetails="response_from_cache"`

  或

  `httpRequest.cacheHit=true`  
  `httpRequest.cacheValidatedWithOriginServer!=true`

* **已向源服务器验证的缓存命中**

  `statusDetails="response_from_cache_validated"`

  或

  `httpRequest.cacheHit=true`  
  `httpRequest.cacheValidatedWithOriginServer=true`

* **缓存未命中**

  `statusDetails="response_sent_by_backend"`

  或

  `httpRequest.cacheHit!=true`  
  `httpRequest.cacheLookup=true`

布尔值类型的日志字段通常仅在其值为 `true` 时才会显示。 如果某个布尔值字段的值为 `false`，则日志中将不会出现该字段。

系统将为这些字段强制执行 [UTF-8](https://wikipedia.org/wiki/UTF-8) 编码。非 UTF-8 字符将被替换为问号。

当 Cloud CDN 通过发起验证请求和/或字节范围请求来处理客户端请求时，它会忽略客户端请求的 Cloud Logging 日志条目中的 `serverIp` 字段。这是因为，Cloud CDN 可能会将请求发送到多个服务器 IP 地址，以便响应单个客户端请求。

Cloud CDN 发起的每个请求都会创建一个 Cloud Logging 日志条目。生成的日志条目在 `jsonPayload` 内包含一个 `parentInsertId` 字段。对于单个客户端请求，您可以使用此字段标识日志条目的 `insertId`，以提示 Cloud CDN 启动验证请求或字节范围请求。此外，日志条目将 Cloud CDN 标识为用户代理。

### 后续步骤 <a id="whats_next"></a>

* 如需详细了解日志记录（包括如何将日志导出到 BigQuery、Pub/Sub 或 Cloud Storage）以及如何配置基于日志的指标以进行监控和提醒，请参阅 [Cloud Logging 文档](https://cloud.google.com/logging/docs)。

