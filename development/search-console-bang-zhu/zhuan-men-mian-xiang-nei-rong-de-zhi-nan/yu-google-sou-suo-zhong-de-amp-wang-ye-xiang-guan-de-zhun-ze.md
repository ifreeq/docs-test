# 与 Google 搜索中的 AMP 网页相关的准则

我们为如何使网站便于 Google 处理而制定的所有[准则](https://support.google.com/webmasters/answer/40349)也都适用于 AMP 网页。本文档将介绍我们专为 Google 搜索中的 AMP 网页而制定的一些其他准则。要详细了解 Google 搜索中的 AMP 网页，请阅读我们的[开发者指南](https://developers.google.com/search/docs/guides/about-amp)。

* 您的 AMP 网页必须遵守 [AMP HTML 规范](https://www.ampproject.org/docs/reference/spec.html)。如果您是新手，请了解如何[创建您的首个 AMP HTML 网页](https://www.ampproject.org/docs/get_started/create.html)。
* 就可浏览的内容和可完成的操作而言，您必须尽可能地让用户能在 AMP 网页上获得与在对应的规范网页上相同的体验。
* 您的 AMP 网址架构应易于用户理解。例如，如果您的规范网页是 `example.com/giraffes`，那么您应将对应的 AMP 网页托管在与 `amp.example.com/giraffes` 或 `example.com/amp/giraffes` 类似的网址上（而不应托管在 `test.com/giraffes` 上）。这是因为当用户在 Google 搜索结果中点击某个指向您的 AMP 网页的链接时，浏览器内即会显示相应的 AMP 网址（就像任何网页一样）；倘若所显示的网址与您的主网站完全不相关，则可能会令用户感到困惑。
* 您的 AMP 网页必须[有效](https://search.google.com/test/amp)，才能按预期向用户呈现，且才能使用与 AMP 相关的功能。包含无效 AMP 标记的网页将无法使用某些搜索功能。
* 如果您向网页中添加结构化数据，请务必遵守我们的[结构化数据政策](https://developers.google.com/structured-data/policies)。

### 常见问题解答

**平板电脑或桌面设备上未显示我的 AMP 专用功能，这是为什么？**

Google 上的 AMP 专用功能（例如“焦点新闻”轮换展示）目前仅适用于移动设备。尽管 AMP 网页本身适用于所有类型的设备（包括桌面设备），但我们目前还没制定将 AMP 专用功能扩展到非移动平台的计划。

**AMP 网页只能在移动设备上显示吗？**

不是。AMP 网页可供在所有类型的设备上查看，因此请使用[自适应设计](https://www.ampproject.org/docs/guides/author-develop/responsive_amp)构建您的 AMP 网页。

**AMP 网页在桌面设备上的呈现效果如何？**

AMP 网页在移动设备屏幕和桌面设备屏幕上的显示效果一样好。如果 AMP 支持您所需的全部功能，您不妨考虑将您的网页创建为[独立的 AMP 网页](https://www.ampproject.org/docs/guides/deploy/discovery#what-if-i-only-have-one-page)，以满足使用桌面设备的访问者和使用移动设备的访问者对同一网页的不同需求。但是，当桌面设备上的 AMP 网页出现在 Google 搜索结果中时，它们无法使用搜索专用功能进行显示。

