# 使用后端存储分区设置 Cloud CDN



Cloud CDN 利用 Google Cloud 全球外部 HTTP\(S\) 负载平衡器来提供路由、运行状况检查和任播 IP 支持。由于全球外部 HTTP\(S\) 负载平衡器可以有[多种后端实例类型](https://cloud.google.com/load-balancing/docs/features#backends)（Compute Engine 虚拟机实例、Google Kubernetes Engine pod、Cloud Storage 存储分区或 Google Cloud 之外的外部源站），因此您可以选择为哪些后端（源站）启用 Cloud CDN。

本设置指南介绍如何创建一个启用了 Cloud CDN 的简单的外部 HTTP\(S\) 负载平衡器。该示例使用以下资源：

* 默认 Virtual Private Cloud \(VPC\) 网络
* 默认网址映射
* 预留的外部 IP 地址
* 作为后端的 Cloud Storage 存储分区
* 一个负载平衡器后端存储分区，用作 Cloud Storage 存储分区的封装容器

后端存储分区支持以下功能：

* 任何[存储类别](https://cloud.google.com/storage/docs/storage-classes)（包括[多区域](https://cloud.google.com/storage/docs/locations#location-mr)存储分区）的 Cloud Storage 存储分区
* 处于 Google 全球边缘的缓存内容的 Cloud CDN 政策

如需了解 Cloud CDN 的工作原理，请参阅 [Cloud CDN 概览](https://cloud.google.com/cdn/docs/overview)。

### 负载平衡器后端 <a id="load_balancer_backends"></a>

外部 HTTP\(S\) 负载平衡器使用网址映射将来自指定网址的流量定向到指定服务。下表汇总了您可以在其中托管内容和服务的后端类型。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8D1F;&#x8F7D;&#x5E73;&#x8861;&#x5668;&#x540E;&#x7AEF;&#x914D;&#x7F6E;</th>
      <th
      style="text-align:left">&#x5178;&#x578B;&#x5185;&#x5BB9;&#x7C7B;&#x578B;</th>
        <th style="text-align:left">&#x540E;&#x7AEF;&#x7C7B;&#x578B;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x540E;&#x7AEF;&#x670D;&#x52A1;</td>
      <td style="text-align:left">&#x52A8;&#x6001;&#xFF08;&#x5982;&#x6570;&#x636E;&#xFF09;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x975E;&#x6258;&#x7BA1;&#x5B9E;&#x4F8B;&#x7EC4;</li>
          <li>&#x6258;&#x7BA1;&#x5B9E;&#x4F8B;&#x7EC4;</li>
          <li>Google Cloud &#x5185;&#x90E8;&#x7684;&#x7F51;&#x7EDC;&#x7AEF;&#x70B9;&#x7EC4;</li>
          <li>Google Cloud &#x5916;&#x90E8;&#x7684;&#x7F51;&#x7EDC;&#x7AEF;&#x70B9;&#x7EC4;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x540E;&#x7AEF;&#x5B58;&#x50A8;&#x5206;&#x533A;</td>
      <td style="text-align:left">&#x9759;&#x6001;&#xFF08;&#x5982;&#x6620;&#x50CF;&#xFF09;</td>
      <td style="text-align:left">
        <ul>
          <li>Cloud Storage &#x5B58;&#x50A8;&#x5206;&#x533A;&#xFF08;&#x5728;&#x672C;&#x9875;&#x4E2D;&#x8BA8;&#x8BBA;&#xFF09;</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### 准备工作 <a id="before_you_begin"></a>

* 如果您要使用 `gcloud` 或 `gsutil` 实用程序，请参阅[《快速入门：使用 `gsutil` 工具》](https://cloud.google.com/storage/docs/quickstart-gsutil)进行安装。
* [设置默认项目](https://cloud.google.com/sdk/gcloud/reference/config/set)。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud 或 gsutil](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#gcloud-%E6%88%96-gsutil)
  1. 在 Google Cloud Console 中，转到**首页**。

     [转到 Google Cloud 首页](https://console.cloud.google.com/home/)

  2. 在 Google Cloud 的右侧，从下拉菜单中选择一个项目。

#### 创建 Cloud Storage 存储分区 <a id="creating_a_bucket"></a>

如果您已有尚未分配给负载平衡器的 Cloud Storage 存储分区，则可以跳至[下一步](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#copy-file)。

创建 Cloud Storage 存储分区以用作使用 Cloud CDN 的外部 HTTP\(S\) 负载平衡器的后端时，我们建议您选择一个[多区域存储分区](https://cloud.google.com/storage/docs/locations#location-mr)，它会自动跨多个 Google Cloud 区域复制对象。这可以提高内容的可用性，并提高应用的故障容忍度。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gsutil](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#gsutil)

1. 在 Google Cloud Console 中打开 **Cloud Storage 浏览器**。

   [打开 Storage 浏览器](https://console.cloud.google.com/storage/browser)

2. 点击**创建存储分区**。
3. 为下表中的字段指定值，其他所有字段保留默认值：

   | 属性 | 值（输入值或选择指定的选项） |
   | :--- | :--- |
   | 名称 | 为每个存储分区输入一个全局唯一的名称。如果您输入的名称不是唯一的，则会看到一条消息，提示您尝试使用其他名称。 |
   | 位置类型 | **多区域** |
   | 位置 | 选择一个区域，例如 **us（美国的多个区域）**。 |
   | 默认存储类别 | **标准** |
   | 访问控制 | **统一** |

4. 点击**创建**。
5. 记下新创建的 Cloud Storage 存储分区的名称，以进行下一步。

#### 将图形文件复制到 Cloud Storage 存储分区 <a id="copy-file"></a>

要使您能够测试设置，请将图形文件从公共 Cloud Storage 存储分区复制到自己的 Cloud Storage 存储分区。

1. 在 Cloud Shell 中运行以下命令。将 `BUCKET_NAME` 替换为您的 Cloud Storage 存储分区的唯一名称：

   ```text
   gsutil cp gs://gcp-external-http-lb-with-bucket/three-cats.jpg gs://BUCKET_NAME/static/us/
   ```

2. 在 Cloud Console 中，点击**刷新**，以验证图形文件已复制。

#### 将 Cloud Storage 存储分区设为公开 <a id="making_your_bucket_public"></a>

此示例使您的 [Cloud Storage 存储分区可供公开读取](https://cloud.google.com/storage/docs/access-control/making-data-public#buckets)。这是公开内容的推荐方法。如此设置时，互联网上的任何人都可以查看和列出您的对象及其元数据（不包括 ACL）。推荐做法是将特定 Cloud Storage 存储分区专用于公开对象。如需了解详情，请参阅[建议的存储分区架构](https://cloud.google.com/storage/docs/access-control#recommended_bucket_architecture)。**重要提示**：从公共 Cloud Storage 存储分区传送对象时，[默认情况下](https://cloud.google.com/storage/docs/metadata#caching_data)，对象应用了 `Cache-Control: public, max-age=3600` 响应标头。这样，您可以在启用 Cloud CDN 时缓存对象。

以下是将 Cloud Storage 存储分区设为公开的其他方法：

* 将单个 Cloud Storage 存储分区设为[可供公开读取](https://cloud.google.com/storage/docs/access-control/making-data-public#objects)。我们不建议采用这种方法。
* 使用[签名网址](https://cloud.google.com/cdn/docs/using-signed-urls)。

以下过程授予所有用户查看 Cloud Storage 存储分区中的对象的权限，使该存储分区可公开读取。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gsutil](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#gsutil)

1. 在 Google Cloud Console 中打开 **Cloud Storage 浏览器**。

   [打开 Storage 浏览器](https://console.cloud.google.com/storage/browser)

2. 导航到该存储分区，然后点击**权限**标签页。
3. 点击**添加成员**。
4. 在**新成员**中输入 `allUsers`。
5. 对于角色，请选择 **Cloud Storage &gt; Storage Object Viewer**。
6. 点击**保存**。

### 预留外部 IP 地址 <a id="ip-address"></a>

现在您的 Cloud Storage 存储分区已启动并正在运行，接下来请设置[全局静态外部 IP 地址](https://cloud.google.com/compute/docs/ip-addresses#externaladdresses)，您的客户将使用此地址访问负载平衡器。

此步骤是可选的，但建议执行此操作，因为静态外部 IP 地址会提供单个地址以指向您的网域。**注意**：您可以跳过此步骤，让 Google Cloud 将临时 IP 地址与负载平衡器的转发规则相关联。转发规则存在时，临时 IP 地址保持不变。如果您需要删除转发规则并重新添加，则转发规则可能会收到新的 IP 地址。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#gcloud)

1. 在 Google Cloud Console 中，转到**外部 IP 地址**页面。

   [转到“外部 IP 地址”页面](https://console.cloud.google.com/addresses/list)

2. 如需预留 IPv4 地址，请点击**预留静态地址**。
3. 指定**名称** `example-ip`。
4. 将**网络服务层级**设置为**优质**。
5. 将 **IP 版本**设置为 **IPv4**。
6. 将**类型**设置为**全局**。
7. 点击**保留**。

### 创建外部 HTTP\(S\) 负载平衡器 <a id="creating_the"></a>

如果要创建 HTTPS 负载平衡器，您必须具有可添加到负载平衡器前端的 SSL 证书资源。如需了解详情，请参阅 [SSL 证书概览](https://cloud.google.com/load-balancing/docs/ssl-certificates)。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)

**开始外部 HTTP\(S\) 负载平衡器配置过程**

1. 在 Google Cloud Console 中，转到**负载平衡**页面。

   [转到“负载平衡”页面](https://console.cloud.google.com/networking/loadbalancing/add)

2. 在 **HTTP\(S\) 负载平衡**下，点击**开始配置**。
3. 选择**从互联网到我的虚拟机**，然后点击**继续**。
4. 将**名称**设置为 `http-lb`，然后转到下一步。

**配置后端并启用 Cloud CDN**

创建负载平衡器的后端存储分区，将其用作 Cloud Storage 存储分区的封装容器。创建或修改后端存储分区时，您可以启用 Cloud CDN。

1. 点击**后端配置**。
2. 在**后端服务和后端存储分区**下，点击**创建或选择后端服务和后端存储分区**，然后点击**后端存储分区 &gt; 创建后端存储分区**。
3. 将**名称**设置为 `backend-bucket1`。 此名称不需要是全局唯一的，并且可以与实际 Cloud Storage 存储分区的名称不同。
4. 在 **Cloud Storage 存储分区**下，点击**浏览**。
5. 选择您创建的 Cloud Storage 全局唯一 `BUCKET_NAME`，然后点击**选择**。
6. 点击**启用 Cloud CDN**。**重要提示**：请确保您在此步骤中选择**启用 Cloud CDN**。忘记执行此操作会导致直接从 Cloud Storage 存储分区传送对象，而不是从 Google 的全球 CDN 缓存传送。
7. 点击**创建**。

**配置主机规则和路径匹配器**

在**主机和路径规则**中，您可以保留默认设置。

如需查看自定义设置示例，请参阅[将后端存储分区添加到负载平衡器](https://cloud.google.com/load-balancing/docs/https/adding-backend-buckets-to-load-balancers)。

如需详细了解主机规则和路径匹配器，请参阅[网址映射概览](https://cloud.google.com/load-balancing/docs/url-map-concepts)。

**配置前端**

1. 点击**前端配置**。
2. 验证下表中的选项配置了这些值。

   | 属性 | 值（输入值或选择指定的选项） |
   | :--- | :--- |
   | 协议 | **HTTP** |
   | 网络服务层 | **高级** |
   | IP 版本 | **IPv4** |
   | IP 地址 | `example-ip` |
   | 端口 | **80** |

   如果您希望创建 HTTPS 负载平衡器而不是 HTTP 负载平衡器，您必须拥有 [SSL 证书](https://cloud.google.com/load-balancing/docs/ssl-certificates) \(`gcloud compute ssl-certificates list`\)，并且必须按如下所示填写相关字段。

   | 属性 | 值（输入值或选择指定的选项） |
   | :--- | :--- |
   | 协议 | **HTTPS** |
   | 网络服务层 | **高级** |
   | IP 版本 | **IPv4** |
   | IP 地址 | `example-ip` |
   | 端口 | **443** |
   | 证书 | **选择证书**或**创建新证书** |

3. 点击**完成**。

**检查配置**

1. 点击**检查并最终确定**。
2. 检查**后端存储分区**、**主机和路径规则**和**前端**部分。
3. 点击**创建**。
4. 等待负载平衡器完成创建。
5. 点击负载平衡器的名称 \(**http-lb**\)。
6. 请记下负载平衡器的 IP 地址，以便在下一个任务中使用。它被称为 `IP_ADDRESS`。

### 向后端存储分区发送流量 <a id="send-traffic"></a>

创建全局转发规则后，您的配置可能需要几分钟时间才能传播到全球范围。几分钟后，您就可以开始将流量发送到负载平衡器的 IP 地址。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#gcloud)

1. 在 Google Cloud Console 中，转到**负载平衡**页面。

   [转到“负载平衡”页面](https://console.cloud.google.com/networking/loadbalancing/)

2. 点击 `http-lb`，展开您刚创建的负载平衡器。

   在**后端**部分中，确认后端存储分区运行状况良好。 您的后端存储分区旁边应该有一个绿色选中标记。否则，请先尝试重新加载页面。Cloud Console 可能需要一段时间才能指示后端状况良好。

3. 在 Cloud Console 显示后端存储分区运行状况良好后，您可以使用网络浏览器转到 `http://IP_ADDRESS/static/us/three-cats.jpg` 来测试您的负载平衡器。将 `IP_ADDRESS` 替换为[负载平衡器的 IP 地址](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#reserving_external_ip_addresses)。您的浏览器应该会打开一个页面，其中包含呈现图形文件的内容。

### 验证 Cloud CDN 是否正常运行 <a id="verifying_that_is_working"></a>

如果您连续多次重新加载 `http://ip-address/static/us/three-cats.jpg` 页面，则应该有多次缓存命中。

以下日志条目显示了缓存命中。您可以通过在 Google Cloud Console 中打开日志查看器并按转发规则名称进行过滤来查看缓存命中。

[打开日志查看器](https://console.cloud.google.com/logs?service=network.googleapis.com)**注意**：如果在设置负载平衡器时未指定转发规则名称，则转发规则名称为负载平衡器名称，再加上 `-forwarding-rule`。在此示例中为 `http-lb-forwarding-rule`。[日志查看器](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%97%A5%E5%BF%97%E6%9F%A5%E7%9C%8B%E5%99%A8)

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
   forwarding_rule_name: "http-lb-forwarding-rule"
   project_id: "PROJECT_ID"
   target_proxy_name: "http-lb-target-proxy"
   url_map_name: "http-lb"
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

### 停用 Cloud CDN <a id="disable"></a>

[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket#gcloud)

**为单个后端存储分区停用 Cloud CDN**

1. 在 Google Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list)

2. 在源站所在行的右侧，点击**菜单** more\_vert，然后选择**修改**。
3. 取消选中您想要停止使用 Cloud CDN 的所有后端存储分区的复选框。
4. 点击**更新**。

**为某个源站的所有后端存储分区移除 Cloud CDN**

1. 在 Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list)

2. 在源站所在行的右侧，点击**菜单** more\_vert，然后选择**移除**。
3. 点击**移除**进行确认。

停用 Cloud CDN 后，系统不会使缓存失效或完全清除缓存。如果您关闭 Cloud CDN，然后重新开启 Cloud CDN，则大部分或所有缓存的内容仍可能会被缓存。为了防止缓存使用内容，您必须[使该内容失效](https://cloud.google.com/cdn/docs/invalidating-cached-content)。

### 后续步骤 <a id="whats_next"></a>

* 如需了解缓存的内容，请参阅[缓存概览](https://cloud.google.com/cdn/docs/caching)。
* 如需在 GKE 中使用 Cloud CDN，请参阅 [Ingress 功能](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-features)。
* 如需检查 Cloud CDN 是否正在从缓存传送响应，请参阅[查看日志](https://cloud.google.com/cdn/docs/logging)。
* 如需了解常见问题及解决方案，请参阅[问题排查](https://cloud.google.com/cdn/docs/troubleshooting-steps)。

