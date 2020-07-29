# 关于我们的统计信息和数据

### 何时显示 Search Console 数据？

当您将网站添加至 Search Console 后，诊断数据和其他数据可能要过一段时间才能显示出来。这属于正常情况，因为 Search Console 可能需要一些时间来收集和处理您网站的数据。一般而言，如果您看到“尚无任何数据”的消息，请稍后再回来查看。一旦 Google 开始更频繁地抓取您的网站，您就会发现 Search Console 开始显示更详细的数据，而且数据更新也会更频繁。

如果您仍然看不到任何数据，则可能是由以下某个原因导致的：

* 如果您没有看到 www.example.com 网站的任何数据，可能是因为您添加网站时用的是 http://example.com 的形式。对于 Google 而言，这两个网站完全不同。如果您觉得缺少某些数据，请将带 www 和不带 www 的网域都添加到您的 Search Console 帐户中，然后看一下这两个版本的网站数据。
* 如果您在**抓取错误**报告中看不到任何数据，则可能是因为 Google 根本就未发现任何问题。这是好消息，但我们建议您定期回来查看。
* 如果您在**搜索分析**报告中看不到任何数据，则可能是因为还没有人在搜索结果中点击您的网站。请查看可能导致[您的网站在搜索结果中表现不佳](https://support.google.com/webmasters/answer/34444)的原因。
* 如果您在**站点地图**报告中看不到任何数据，请考虑[创建和提交站点地图](https://support.google.com/webmasters/answer/183668)，以帮助 Google 发掘您网站上的内容。

我们一直在努力提高已验证网站的数据（如抓取、索引及搜索分析统计信息）的更新频率。这些数据大部分基于您的网站内容，因此最能反映出您的网站现状。我们的内部系统一直在变化，而网络本身也是一个不断改变的环境。另外，从算出数目到网站站长看到该数目，这中间可能会出现延迟。虽然我们会间隔一定时间来发布数据，但收集数据的工作却是持续进行的。如果您的内容不经常变化，或者如果您没有增加指向自己网站的新链接，则您可能不会在每次登录 Search Console 时都看到数据更新。

### 如果数据缺失或已过期

当您将网站添加至 Search Console 后，诊断数据和其他数据需要过一段时间才能显示出来。

如果您 1 周后仍没看到新网站 www.example.com 的任何数据，可能是因为您添加网站时用的是 http://example.com 的形式。对于 Search Console 而言，example.com 和 www.example.com 是两个完全不同的网站。如果您觉得缺少某些数据，请将您网域的 www 版本和非 www 版本都添加到 Search Console 帐号中，然后看一下这两个版本的网站数据。

对不带 www 的网域执行“site:”搜索（例如，\[site:example.com\]）。这应当会返回您域中的网页以及编入索引的所有子域（例如 www.example.com、rollergirl.example.com 等）。您应当能够从结果中看出，编入索引的网站主要是带 www 子域名的网站还是没有 www 子域名的网站。编入索引的版本可能是在您的 Search Console 帐号中显示了最多数据的版本。 [详细了解如何指定网页的规范版本](https://support.google.com/webmasters/answer/139066)。

### 为什么 Search Console 数据与 Google Analytics（分析）数据不一致？

Search Console 数据可能会与其他工具（如 [Google Analytics（分析）](http://www.google.com/analytics)）中显示的数据有所不同。可能的原因包括：

* 在您的网站上次更改后，我们可能没有对其进行抓取。
* Search Console 会进行其他一些数据处理工作（例如，从漫游器中排除重复内容和访问次数），这可能导致您的统计信息与其他来源中所列的统计信息不符。
* 某些工具（例如 Google Analytics（分析））只跟踪已在浏览器中启用 JavaScript 的用户所产生的流量。

