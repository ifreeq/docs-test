---
description: 保护您的网站和用户
---

# 使用 HTTPS 确保网站安全

### 什么是 HTTPS？

HTTPS（超文本传输安全协议）是一种互联网通信协议，可确保在用户的计算机与网站之间所传递的数据的完整性和机密性。每当访问网站时，用户都希望自己的在线体验安全无虞且具有私密性。因此，我们建议您采用 HTTPS 来保护用户与您网站之间的连接（无论您网站上提供了什么内容）。

所有使用 HTTPS 发送的数据都可通过传输层安全协议 \([TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)\) 得到保护。该协议可提供三重关键保护：

1. **加密** - 对所交换的数据进行加密，以使其免受窥探。这意味着在用户浏览网站期间，没有人能够“听到”其会话内容，也无法在多个网页上跟踪其活动或窃取其信息。
2. **数据完整性** - 不管是有意还是无意，在数据传输期间数据都无法被修改或损坏，也不会被检测。
3. **身份验证** - 证明用户可与目标网站通信，这有助于保护用户免遭[中间人攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)并建立用户信任，进而带来其他商业效益。

### 实施 HTTPS 时的最佳做法

#### 使用强大的安全证书

在为网站启用 HTTPS 的过程中，您必须获得安全证书。证书由[数字证书认证机构](https://en.wikipedia.org/wiki/Certificate_Authority) \(CA\) 颁发，该机构会采取有关措施，确认您的网站地址是否确实属于您的组织，从而保护访问者免受中间人攻击。在设置证书时，您可以选择 2048 位密钥，来确保高级别的安全性。如果所持证书的密钥（1024 位）安全性较弱，请将其升级到 2048 位。选择网站证书时，请注意以下几点：

* 从提供技术支持的可靠 CA 处获取证书。
* 确定所需证书的类型：
  * 适用于单个安全源的单个证书（例如：`www.example.com`）。
  * 适用于多个已知安全源的多网域证书（例如`www.example.com、cdn.example.com、example.co.uk`）。
  * 适用于具有多个动态子域名的安全源的通配型证书（例如：`a.example.com、b.example.com`）。

#### 使用服务器端 301 重定向

使用服务器端 301 HTTP 重定向将用户和搜索引擎重定向至 HTTPS 网页或资源。

#### 确认 Google 能否抓取您的 HTTPS 网页并将其编入索引

* 请勿通过 robots.txt 文件阻止抓取您的 HTTPS 网页。
* 请勿在您的 HTTPS 网页中包含 `noindex` 元标记。
* 使用[“网址检查”工具](https://support.google.com/webmasters/answer/9012289)来测试 Googlebot 能否访问您的网页。

#### 支持 HSTS

我们建议 HTTPS 网站支持 HSTS（[HTTP 严格传输安全协议](https://zh.wikipedia.org/wiki/HTTP_Strict_Transport_Security)）。HSTS 不仅会告知浏览器自动请求 HTTPS 页面（即使用户在浏览器地址栏中输入的是 `http`），还会告知 Google 在搜索结果中提供安全网址，从而最大限度地降低了向用户提供不安全内容的风险。

要支持 HSTS，请使用支持 HSTS 的网络服务器并启用该功能。

虽然 HSTS 更安全，但它会增加回滚策略的复杂性。我们建议您按照以下方式启用 HSTS：

1. 首先，滚动未启用 HSTS 的 HTTPS 页面。
2. 开始发送 max-age 较短的 HSTS 标头。通过用户和其他客户端监控您的流量，并监控相关内容的效果，如广告。
3. 逐步增加 HSTS 的 max-age。
4. 如果 HSTS 不会对您的用户和搜索引擎产生负面影响，您便可请求系统将您的网站添加到被大多数主要浏览器使用的 [HSTS 预加载列表](https://hstspreload.org/)中（如果您愿意的话）。

**考虑使用 HSTS 预加载**

如果您启用了 HSTS，则可选择支持 [HSTS 预加载](https://www.google.com/webhp?#q=about+hsts+preloading)，以进一步提高安全性并改善性能。要启用预加载，您必须[访问 hstspreload.org 并按照其中所述的提交要求对您的网站执行相应操作](https://hstspreload.org/)。

#### 避免以下常见问题 <a id="pitfalls"></a>

在使用 TLS 保护网站安全的整个过程中，请避免以下错误：

| **问题** | **措施** |
| :--- | :--- |
| 过期的证书 | 确保证书始终最新。 |
| 注册了错误网站名称的证书 | 检查您是否已获得可涵盖与您网站对应的所有主机名的证书。例如，如果您的证书仅涵盖 www.example.com，则仅使用 example.com（不带“www.”前缀）加载您的网站的访问者将会因证书名称不匹配错误而被禁止访问。 |
| 缺少[服务器名称指示](https://en.wikipedia.org/wiki/Server_Name_Indication#Support) \(SNI\) 支持 | 确保网络服务器支持SNI且您的受众通常使用支持的浏览器。所有[新式浏览器](https://en.wikipedia.org/wiki/Server_Name_Indication#Support)都支持SNI，但如果您需要支持旧版浏览器，则需要一个专用的IP。 |
| 抓取问题 | 请勿使用`robots.txt`屏蔽抓取HTTPS网站。 |
| 索引编制问题 | 尽可能允许通过搜索引擎将网页编入索引。避免使用 `noindex` 元标记。 |
| 旧版协议 | 旧版协议易受攻击；请务必使用最新版 TLS 库并实施最新版协议。 |
| 并非只含安全元素 | 在 HTTPS 网页上只嵌入 HTTPS 内容。 |
| HTTP 和 HTTPS 上的内容不同 | 确保HTTP网站和HTTPS上的内容相同。 |
| HTTPS上的[HTTP状态代码](https://support.google.com/webmasters/answer/40132?hl=zh-CN)错误 | 确认网站是否返回正确的HTTP状态代码。例如，`200 OK`（页面可访问）或 `404`/`410`（页面不存在）。 |

#### 更多提示

有关在网站上使用 HTTPS 网页的更多提示，请参阅 [HTTPS 迁移常见问题解答](https://support.google.com/webmasters/answer/6033049#https-faqs)。

### 从 HTTP 迁移到 HTTPS

如果您将网站从 HTTP 迁移到 HTTPS，则 Google 只会将其视为[包含网址更改的网站迁移](https://support.google.com/webmasters/answer/6033049)。这可能会暂时影响您的部分流量。有关详情，请访问[网站迁移概览页面](https://support.google.com/webmasters/answer/34437)。

**在 Search Console 中添加新的 HTTPS 资源**：Search Console 会分别处理 HTTP 和 HTTPS；数据不会在 Search Console 中的资源之间共享。

请参阅[站点地图迁移的问题排查页面](https://support.google.com/webmasters/answer/6033088)，以排查迁移问题。

### 更多信息

要详细了解如何在您的网站上实施 TLS，请参阅以下资源：

* [Qualys SSL/TLS 最佳做法](https://www.ssllabs.com/projects/documentation/)
* [SSL/TLS Mozilla wiki](https://wiki.mozilla.org/Security/Server_Side_TLS)

