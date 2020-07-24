# 使用代管实例组设置 Cloud CDN



Cloud CDN 利用 Google Cloud 全球外部 HTTP\(S\) 负载平衡器来提供路由、运行状况检查和任播 IP 支持。全球外部 HTTP\(S\) 负载平衡器可以有[多个后端实例类型](https://cloud.google.com/load-balancing/docs/features?authuser=1#backends)，并且您可以选择要为哪些后端（也称为源站）启用 Cloud CDN。本设置指南介绍如何创建一个启用了 Cloud CDN 的简单的外部 HTTP 负载平衡器。负载平衡器具有以下资源：

* 默认 Virtual Private Cloud \(VPC\) 网络
* Compute Engine 托管实例组
* 默认网址映射
* 预留的外部 IP 地址

如需设置具有“TLS 终止”功能的简单外部 HTTPS 负载平衡器，请参阅[设置简单的外部 HTTPS 负载平衡器](https://cloud.google.com/load-balancing/docs/https/ext-https-lb-simple?authuser=1)。

如需查看包含 IPv6 和 SSL 证书设置的基于内容的多区域示例，请参阅[设置基于内容的多区域外部 HTTPS 负载平衡器](https://cloud.google.com/load-balancing/docs/https/setting-up-https?authuser=1)。

如需了解一般概念，请参阅[外部 HTTP\(S\) 负载平衡概览](https://cloud.google.com/load-balancing/docs/https?authuser=1)。

如果您使用的是 GKE，则通常由 Kubernetes Ingress 控制器配置负载平衡器。有关详情，请参阅[配置 Ingress 以进行外部负载平衡](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress?authuser=1)。**注意**：如果您已经拥有包含一个或多个实例组的外部 HTTP\(S\) 负载平衡器，则可以跳至本指南的[启用 Cloud CDN](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#enable-cdn) 部分。

### 拓扑 <a id="topology"></a>

在本指南中，您将创建下图所示的配置。[![&#x7B80;&#x5355;&#x7684; HTTP &#x8D1F;&#x8F7D;&#x5E73;&#x8861;&#xFF08;&#x70B9;&#x51FB;&#x53EF;&#x653E;&#x5927;&#xFF09;](https://cloud.google.com/load-balancing/images/http-load-balancer-simple.svg?authuser=1)](https://cloud.google.com/load-balancing/images/http-load-balancer-simple.svg?authuser=1)简单的 HTTP 负载平衡（点击可放大）

图中的事件顺序如下所示：

1. 客户端向转发规则中定义的外部 IPv4 地址发送内容请求。
2. 负载平衡器检查是否可以从缓存中处理请求。如果可以，负载平衡器将请求的内容从缓存中传送出去。否则，继续进行处理。
3. 转发规则将请求定向到目标 HTTP 代理。
4. 目标代理使用网址映射中的规则确定单个后端服务接收所有请求。
5. 负载平衡器确定后端服务只有一个实例组，并将请求定向到该组中的虚拟机实例。
6. 该虚拟机提供用户请求的内容。

[![&#x542F;&#x7528;&#x4E86; Cloud CDN &#x7684;&#x7B80;&#x5355; HTTP\(S\) &#x8D1F;&#x8F7D;&#x5E73;&#x8861;&#xFF08;&#x70B9;&#x51FB;&#x53EF;&#x653E;&#x5927;&#xFF09;](https://cloud.google.com/cdn/images/cdn-response-flow-simple.svg?authuser=1)](https://cloud.google.com/cdn/images/cdn-response-flow-simple.svg?authuser=1)启用了 Cloud CDN 的简单 HTTP\(S\) 负载平衡（点击可放大）

### 权限 <a id="permissions_2"></a>

为完成本指南中的步骤，您必须拥有在项目中创建 Compute Engine 实例、防火墙规则和预留 IP 地址的权限。您必须拥有项目的 [Owner 或 Editor 角色](https://cloud.google.com/iam/docs/understanding-roles?authuser=1#primitive_roles)，或者必须拥有以下 [Compute Engine IAM 角色](https://cloud.google.com/compute/docs/access/iam?authuser=1)。

| 任务 | 所需角色 |
| :--- | :--- |
| 创建实例 | [Instance Admin](https://cloud.google.com/compute/docs/access/iam?authuser=1#compute.instanceAdmin) |
| 添加和移除防火墙规则 | [Security Admin](https://cloud.google.com/compute/docs/access/iam?authuser=1#compute.securityAdmin) |
| 创建负载平衡器组件 | [Network Admin](https://cloud.google.com/compute/docs/access/iam?authuser=1#compute.networkAdmin) |
| 创建项目（可选） | [Project Creator](https://cloud.google.com/resource-manager/docs/creating-managing-projects?authuser=1#creating_a_project) |

如需了解详情，请参阅以下指南：

* [访问权限控制](https://cloud.google.com/load-balancing/docs/access-control?authuser=1)
* [Cloud IAM Conditions](https://cloud.google.com/load-balancing/docs/access-control/iam-conditions?authuser=1)

### 创建托管实例组 <a id="mig"></a>

如需使用 Compute Engine 后端设置负载平衡器，您的虚拟机必须属于某个实例组。本指南介绍如何创建由一组运行 Apache 的 Linux 虚拟机组成的代管实例组，然后设置负载平衡。这个托管实例组提供了一组虚拟机，用于运行外部 HTTP 负载平衡器的后端服务器。出于演示目的，后端会传送其各自的主机名。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到**实例组**页面。

   [转到“实例组”页面](https://console.cloud.google.com/compute/instanceGroups?authuser=1)

2. 点击**创建实例组**。
3. 在左侧选择**新建代管实例组**。
4. 对于**名称**，输入 `lb-backend-example`。
5. 在**位置**下方，选择**单个地区**。
6. 对于**区域**，选择您的首选区域。本示例使用 `us-east1`。
7. 对于**区域**，选择 **us-east1-b**。
8. 在**实例模板**下，选择**创建新的实例模板**。
9. 对于**名称**，输入 `lb-backend-template`。
10. 确保启动磁盘已设置为 Debian 映像，例如 **Debian GNU/Linux 9 \(stretch\)**。本文中的说明使用仅 Debian 支持的命令，例如 `apt-get`。
11. 在**管理、安全、磁盘、网络、单独租用**下的**管理**标签页上，将以下脚本插入到**启动脚本**字段。

    ```text
    #! /bin/bash
    apt-get update
    apt-get install apache2 -y
    a2ensite default-ssl
    a2enmod ssl
    vm_hostname="$(curl -H "Metadata-Flavor:Google" \
    http://169.254.169.254/computeMetadata/v1/instance/name)"
    echo "Page served from: $vm_hostname" | \
    tee /var/www/html/index.html
    ```

12. 在**网络**标签页上，添加网络标记 `allow-health-check`。
13. 点击**保存并继续**。
14. 在**自动扩缩模式**下，选择**不自动调节**。
15. 在**实例数**下，输入 `2`。
16. 如需创建新的实例组，请点击**创建**。

**注意**：由于外部 HTTP\(S\) 负载平衡器是代理，因此您不需要在**防火墙**下选择**允许 HTTP 流量**。在[配置防火墙规则](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#firewall)中，您将为此负载平衡器创建唯一必要的防火墙规则。

### 配置防火墙规则 <a id="firewall"></a>

在此示例中，您将创建 `fw-allow-health-check` 防火墙规则。 这是一种入站流量规则，允许来自 Google Cloud 运行状况检查系统（`130.211.0.0/22` 和 `35.191.0.0/16`）的流量。此示例使用目标标记 `allow-health-check` 来标识虚拟机。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到**防火墙**页面。

   [转到“防火墙”页面](https://console.cloud.google.com/networking/firewalls/list?authuser=1)

2. 点击**创建防火墙规则**以创建第二条防火墙规则。
3. 对于**名称**，输入 `fw-allow-health-check`。
4. 在**网络**下，选择 **默认**。
5. 在**目标**下，选择**指定的目标标记**。
6. 使用 `allow-health-check` 填充**目标标记**字段。
7. 将**来源过滤条件**设置为 **IP ranges**。
8. 将**来源 IP 地址范围**设置为 `130.211.0.0/22` 和 `35.191.0.0/16`。
9. 在**协议和端口**下，选择**指定的协议和端口**。
10. 选中 **tcp** 复选框，然后输入端口号 `80`。
11. 点击**创建**。

### 预留外部 IP 地址 <a id="ip-address"></a>

现在您的实例已启动并正在运行，接下来请设置客户用来访问您的负载平衡器的[全局静态外部 IP 地址](https://cloud.google.com/compute/docs/ip-addresses?authuser=1#externaladdresses)。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到**外部 IP 地址**页面。

   [转到“外部 IP 地址”页面](https://console.cloud.google.com/addresses/list?authuser=1)

2. 如需预留 IPv4 地址，请点击**预留静态地址**。
3. 对于**名称**，输入 `lb-ipv4-1`。
4. 将**网络服务层级**设置为**优质**。
5. 将 **IP 版本**设置为 **IPv4**。
6. 将**类型**设置为**全局**。
7. 点击**保留**。

### 设置负载平衡器 <a id="load-balancer"></a>

[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#gcloud)

1. 在 Google Cloud Console 中，转到**负载平衡**页面。

   [转到“负载平衡”页面](https://console.cloud.google.com/networking/loadbalancing/?authuser=1)

2. 点击**创建负载平衡器**。
3. 在 **HTTP\(S\) 负载平衡**下，点击**开始配置**。
4. 选择**从互联网到我的虚拟机**，然后点击**继续**。
5. 在负载平衡器**名称**中输入 `web-map-http`。
6. 点击**后端配置**。
7. 在**创建或选择后端服务和后端存储分区**下，选择**后端服务 &gt; 创建后端服务**。
8. 为您的后端服务添加名称，例如 `web-backend-service`。
9. 在**协议**下，选择 **HTTP**。
10. 在**后端 &gt; 新后端 &gt; 实例组**中，选择您的实例组 `lb-backend-example`。
11. 保留其他默认设置。
12. 在**运行状况检查**下，选择**创建运行状况检查**，然后为您的运行状况检查添加一个名称，例如 `http-basic-check`。
13. 将协议设置为 **HTTP**，然后点击**保存并继续**。
14. 选择**启用 Cloud CDN**。
15. 保留其他默认设置。
16. 点击**创建**。

在**主机和路径规则**中，保留默认设置。

在**前端配置**中，使用以下值：

1. 将**协议**设置为 **HTTP**。
2. 将 **IP 地址**设置为您之前创建的 `lb-ipv4-1`。
3. 确保将**端口**设置为 **80** 以允许 HTTP 流量。
4. 点击**完成**。

点击**检查并最终确定**。

配置完负载平衡器后，点击**创建**。

等待负载平衡器完成创建。

点击负载平衡器的名称。

在**负载平衡器详情**屏幕上，记下负载平衡器的 **IP:端口**。

### 启用 Cloud CDN <a id="enable-cdn"></a>

如果您在创建后端服务时尚未启用 Cloud CDN，请立即通过更新后端服务执行此操作：

```text
gcloud compute backend-services update-backend web-backend-service \
    --enable-cdn
```

**重要提示**：要使 Cloud CDN 缓存来自后端的响应，您需要为每个响应设置有效的缓存指令。例如 `Cache-Control: max-age=3600, public`。

您可以在[缓存概览](https://cloud.google.com/cdn/docs/caching?authuser=1)中详细了解 Cloud CDN 能够识别的缓存指令以及 Cloud CDN 无法缓存的内容。

### 将流量发送到您的实例 <a id="send-traffic"></a>

现在负载平衡服务已运行，您可以将流量发送到转发规则并会发现流量被分散到不同的实例。[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)

1. 在 Google Cloud Console 中，转到**负载平衡**页面。

   [转到“负载平衡”页面](https://console.cloud.google.com/networking/loadbalancing/?authuser=1)

2. 点击您刚刚创建的负载平衡器。
3. 在**后端**部分中，确认虚拟机运行状况良好。**运行状况良好**列应该会填充相应信息，指示两个虚拟机运行状况都良好 \(`2/2`\)。否则，请先尝试重新加载页面。Cloud Console 可能需要一些时间才能指示虚拟机运行状况良好。如果几分钟后后端似乎仍运行状况不佳，请检查防火墙配置以及分配给后端虚拟机的网络标记。
4. 在 Cloud Console 显示后端实例运行状况良好后，您可以使用网络浏览器转到 `http://IP_ADDRESS` 来测试您的负载平衡器。将 `IP_ADDRESS` 替换为[负载平衡器的 IP 地址](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#ip-address)。
5. 您的浏览器应该会呈现一个页面，其中的内容显示提供该页面的实例的名称以及其地区（例如，`Page served from: lb-backend-example-xxxx`）。如果您的浏览器未呈现此页面，请查看本指南中的配置设置。

### 停用 Cloud CDN <a id="disable"></a>

[控制台](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#%E6%8E%A7%E5%88%B6%E5%8F%B0)[gcloud](https://cloud.google.com/cdn/docs/setting-up-cdn-with-mig?authuser=1#gcloud)

**为单个后端服务停用 Cloud CDN**

1. 在 Google Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list?authuser=1)

2. 在源站所在行的右侧，点击**菜单** more\_vert，然后选择**修改**。
3. 取消选中您想要停止使用 Cloud CDN 的所有后端服务的复选框。
4. 点击**更新**。

**为某个源站的所有后端服务移除 Cloud CDN**

1. 在 Cloud Console 中，转到 **Cloud CDN** 页面。

   [转到 Cloud CDN 页面](https://console.cloud.google.com/networking/cdn/list?authuser=1)

2. 在源站所在行的右侧，点击**菜单** more\_vert，然后选择**移除**。
3. 点击**移除**进行确认。

停用 Cloud CDN 后，不会使缓存失效或完全清除缓存。如果您停用然后重新启用 Cloud CDN，则大部分或所有缓存的内容仍可能会被缓存。为了防止内容通过缓存传送，您必须[使该内容失效](https://cloud.google.com/cdn/docs/invalidating-cached-content?authuser=1)。

### 后续步骤 <a id="whats_next"></a>

* 如需了解缓存的内容，请参阅[缓存概览](https://cloud.google.com/cdn/docs/caching?authuser=1)。
* 如需在 GKE 中使用 Cloud CDN，请参阅 [Ingress 功能](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-features?authuser=1)。
* 如需检查 Cloud CDN 是否正在从缓存传送响应，请参阅[查看日志](https://cloud.google.com/cdn/docs/logging?authuser=1)。
* 如需了解常见问题及解决方案，请参阅[问题排查](https://cloud.google.com/cdn/docs/troubleshooting-steps?authuser=1)。
* 如需了解 Cloud CDN 的工作原理，请参阅 [Cloud CDN 概览](https://cloud.google.com/cdn/docs/overview?authuser=1)。

