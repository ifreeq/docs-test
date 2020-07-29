---
description: 了解 Google 如何发现、抓取和呈现网页
---

# Google 搜索的工作方式

Google 的工作方式是什么？下文将依次提供简要版和详尽版的回答。

Google 会从很多不同的来源获取信息，包括：

* 网页；
* 用户提交的内容，例如“Google 我的商家”和地图用户提交的内容；
* 图书扫描；
* 互联网上的公共数据库；
* 以及许多其他来源。

但是，此页面内容将重点介绍 Google 如何通过网页获取信息。

### 简要版

Google 按照以下三个基本步骤来生成基于网页的结果：

#### 抓取

第一步是找出网络上存在哪些网页。不存在包含所有网页的中央注册表，因此 Google 必须不断搜索新网页并将其添加到已知网页列表中。由于 Google 之前已经访问过某些网页，因此这些网页是 Google 已知的网页。当跟踪已知网页上指向新网页的链接时，Google 会发现其他网页。当网站所有者以列表形式（[站点地图](https://support.google.com/webmasters/answer/156184)）提交一系列网页供 Google 抓取时，Google 也会发现其他网页。如果您使用受管的网站托管服务，如 Wix 或 Blogger，这些服务可能会让 Google 抓取您更新后的网页或添加的新网页。

Google 发现网页网址后，会访问或抓取该网页以了解其中的内容。Google 会呈现该网页，并分析文字和非文字内容以及整体视觉布局，确定该网页应显示在搜索结果中的什么位置。Google 越了解您的网站，就越能准确地将您的网站与正在查找您内容的用户相匹配。

**如何改善网站抓取效果：**

* 验证 Google 能否访问您网站上的网页，以及这些网页看起来是否正确。确保 Google 能以匿名用户（没有密码和信息的用户）的身份访问网页。Google 还应该能够查看该网页的所有图片和其他元素，以便能够正确了解该网页。您可以在[移动设备适合性测试工具](https://search.google.com/test/mobile-friendly)中输入网页网址快速检查网页。
* 如果您创建或更新了单个网页，您可以[向 Google 提交具体网址](https://support.google.com/webmasters/answer/6065812)。如需让 Google 同时了解多个新网页或更新后的网页，您可以使用[站点地图](https://support.google.com/webmasters/answer/156184)。
* **如果您想让 Google 只抓取 1 个网页，请将该网页设为首页**。在 Google 看来，您的首页就是您网站上最重要的网页。为促成完整网站抓取，请确保您的首页（以及所有网页）包含一个良好的网站导航系统，能链接到您网站上的所有重要版块和网页。这会有助于用户（和 Google）在您的网站上找到所需内容。对于小型网站（少于 1000 个网页），只需让 Google 知道您的首页即可，前提是 Google 可以通过从首页开始的链接路径访问所有其他网页。
* 将您的网页链接到 Google 已知的其他网页。**但是**，请务必注意，Google 不会跟踪广告中的链接、其他网站中由您付费的链接、评论中的链接或其他未遵循 [Google 网站站长指南](https://support.google.com/webmasters/answer/35769)的链接。

 您无法通过向 Google 付费来提高网站抓取频率或网站排名。任何关于 Google 会在收取费用后提高网站抓取频率的消息均是子虚乌有。

#### 编入索引

发现网页后，Google 会尝试了解该网页的内容。此过程称为“编入索引”。Google 会分析该网页的内容、为网页上嵌入的图片和视频文件编制目录，并通过其他方式尝试了解网页。这些信息存储在 Google 索引中，而 Google 索引是一个存储在很多很多计算机中的巨大数据库。

**如何改善网页索引编制效果：**

* 制作简短且有意义的网页标题。
* 使用传达网页主题的网页标题。
* 使用文字（而非图片）传达内容。（尽管 Google 能够理解一些图片和视频，但相比图片和视频，文字更易于理解。请至少使用替代文本和其他属性为[视频](https://developers.google.com/webmasters/videosearch/)和[图片](https://support.google.com/webmasters/answer/114016)添加适当的注释。）

#### 呈现（和排名）

当用户输入查询时，Google 会根据许多因素尝试从其索引中找到最相关的答案。Google 会努力确定最优质的答案，并会考虑其他因素（例如，考虑用户所在位置、使用的语言及设备（桌面设备或手机）等因素），以便提供最佳用户体验和最恰当的答案。例如，在用户搜索“自行车维修店”后，Google 向巴黎用户显示的答案与向香港用户显示的答案有所不同。Google 不会通过收取费用来提高网页排名，网页排名是以编程方式完成的。

**要改善您的网页呈现和排名效果，请注意以下事项：**

* 提高网页加载速度，并使其适合移动设备访问。
* 在网页上发布实用的内容并保持更新。
* 遵循 [Google 网站站长指南](https://support.google.com/webmasters/answer/35769)，这有助于提供良好的用户体验。
* 详细了解[搜索引擎优化 \(SEO\) 新手指南](https://support.google.com/webmasters/answer/7451184)中的提示和最佳做法。
* 您可以[点击此处以了解详情](https://www.google.com/search/howsearchworks/)，包括[我们为确保提供理想结果而制定的质量评分者指南](https://static.googleusercontent.com/media/www.google.com/en//insidesearch/howsearchworks/assets/searchqualityevaluatorguidelines.pdf)

### 详尽版

#### 抓取

抓取是指 [Googlebot](https://support.google.com/webmasters/answer/answer.py?answer=182072) 访问要添加到 Google 索引中的新网页和更新后的网页的过程。

我们使用大量计算机提取（或“抓取”）网络上的数十亿个网页。执行抓取任务的程序叫做 Googlebot（也称为漫游器或“蜘蛛”程序）。Googlebot 使用算法流程确定要抓取的网站、抓取频率以及要从每个网站抓取的网页数量。

Google 首先会根据一份网页网址列表开始其抓取过程，该列表是在之前进行的抓取过程中生成的，且随着网站站长所提供的站点地图数据的增多而不断扩大。Googlebot 在访问每个网页时，会查找每个网页上的链接，并将这些链接添加到它要抓取的网页的列表中。它会记录新建立的网站、对现有网站进行的更改以及无效链接，并据此更新 Google 索引。

在抓取过程中，Google 会使用 Chrome 的最新版本呈现网页。在呈现过程中，它会运行找到的所有网页脚本。如果您的网站使用动态生成的内容，请务必[遵循 JavaScript SEO 基础知识页面上的要求](https://developers.google.com/search/docs/guides/javascript-seo-basics)。**主要抓取/辅助抓取**

Google 使用两种不同的抓取工具抓取网站：移动版抓取工具和桌面版抓取工具。每种抓取工具类型都会使用该类型的设备模拟访问您网页的用户。

Google 使用 1 种抓取工具类型（移动版或桌面版）作为网站的主要抓取工具。网站上被 Google 抓取的所有网页都是使用主要抓取工具抓取的。对所有新网站使用的主要抓取工具都是移动版抓取工具。

此外，Google 还会使用其他类型的抓取工具（移动版或桌面版）重新抓取网站上的一些网页。这称为辅助抓取，目的在于了解其他设备类型对您网站的适用情况。

**Google 如何得知哪些网页无法抓取？**

* robots.txt 中屏蔽的网页无法抓取，但如果这些网页链接到其他网页，系统仍可能会将其编入索引。（Google 可以通过指向相应网页的链接来推断页面内容，并且在不解析其内容的情况下将相应网页编入索引。）
* Google 无法抓取任何匿名用户无法访问的网页。因此，任何登录或其他授权防护措施都将阻止 Google 抓取网页。
* Google 不会频繁地抓取先前已被抓取且被视为[与其他网页重复](https://support.google.com/webmasters/answer/139066)的网页。

**改善抓取质量**

您可以利用以下这些技巧帮助 Google 发现您网站上的正确网页：

* [提交站点地图](https://support.google.com/webmasters/answer/156184)。
* [提交单个网页的抓取请求](https://support.google.com/webmasters/answer/6065812)。
* 针对网页使用[简单易懂的逻辑网址路径](https://support.google.com/webmasters/answer/76329)，并在网站中提供清晰直接的内部链接。
* 如果您在网站上使用网址参数进行导航，例如，如果您在全球购物网站上指明用户所在的国家/地区，请[使用网址参数工具告知 Google 关于重要参数的信息](https://support.google.com/webmasters/answer/6080550)。
* 谨慎使用 robots.txt：使用 robots.txt 指明您希望 Google 优先了解或抓取哪些网页，从而降低服务器负载，请勿将其作为阻止材料出现在 Google 索引中的方法。
* 使用 [hreflang](https://support.google.com/webmasters/answer/189077) 指向其他语言版本的网页。
* 明确指出[规范网页和备用网页](https://support.google.com/webmasters/answer/139066)。
* 通过[“索引涵盖范围”报告](https://support.google.com/webmasters/answer/7440203)查看您的抓取和索引涵盖范围。
* 确保 Google 可以访问主要网页以及正确呈现网页所需的重要资源（图片、CSS 文件、脚本）。
* 用[网址检查工具](https://support.google.com/webmasters/answer/9012289)检查实际网页，确认 Google 可以正常访问并呈现您的网页。

#### 编入索引 <a id="indexing_advanced"></a>

Googlebot 会处理它抓取的每个网页，以便了解每个网页的内容。这包括处理文字内容、关键内容标记和属性，例如 `<title>` 标记和 Alt 属性、图片、视频等。Googlebot 可处理多种类型的内容，但并不是所有类型的内容都能处理。例如，我们无法处理某些富媒体文件的内容。

在抓取和编入索引的间隙，Google 会确定网页是否是另一网页的[重复网页或规范网页](https://support.google.com/webmasters/answer/139066)。如果该网页被视为重复网页，Google 便会显著降低对它的抓取频率类似网页会归入一个文档中，其中列出了一个或多个网页，包括规范网页（这组网页中最具代表性的网页）和找到的所有重复网页（可能只是访问同一网页的备用网址，或者可能是同一网页的备用移动版或桌面版）。

请注意，Google 不会将包含 [noindex 指令](https://support.google.com/webmasters/answer/93710)（标头或标记）的网页编入索引。但前提是 Google 必须能够看到该指令；如果网页被 [robots.txt 文件](https://support.google.com/webmasters/answer/6062608)、登录页或其他设备屏蔽了，那么即使 Google 并未访问该网页，也可能会将其编入索引！

**改善编入索引的效果**

您可以通过多种技巧使 Google 更加了解您的网页内容：

* 使用 [noindex](https://support.google.com/webmasters/answer/93710) 阻止 Google 抓取或找到您要隐藏的网页。请勿对 robots.txt 屏蔽的网页添加“noindex”；如果您这样做，Google 将看不见“noindex”指令，并且仍会将该网页编入索引。
* [使用结构化数据](https://developers.google.com/search/docs/guides/intro-structured-data)。
* 遵循 [Google 网站站长指南](https://support.google.com/webmasters/answer/35769)。
* 查看[基本 SEO 指南](https://support.google.com/webmasters/answer/7451184)和[高级用户指南](https://support.google.com/webmasters/answer/9429907)，了解更多提示。

**什么是“文档”？**

Google 在内部将网页表示为大量文档。每个文档都表示一个或多个网页。这些网页完全相同或非常相似但本质上内容相同，可以通过不同网址访问。文档中的不同网址可能会指向完全相同的网页（例如，example.com/dresses/summer/1234 和 example.com?product=1234 可能会显示同一网页），或同一网页对使用不同设备的用户来说具有细微差别（例如，example.com/mypage 适合桌面设备用户，m.example.com/mypage 适合移动设备用户）。

Google 会从文档中选择 1 个网址，并将其定义为该文档的[规范](https://support.google.com/webmasters/answer/139066)网址。文档的规范网址是 Google 最常抓取和编入索引的网址；其他网址会被视为重复网址或备用网址，并且[可能会偶尔被抓取](https://support.google.com/webmasters/answer/182072)，或根据用户请求将其作为结果呈现：例如，如果文档的规范网址是移动网址，Google 仍可能会为用桌面设备搜索的用户提供桌面（备用）网址。

Search Console 中的大多数报告都会将数据归到文档的规范网址名下。某些工具（例如“检查网址”工具）支持测试备用网址，但检查规范网址也应提供有关备用网址的信息。

您可以[告知 Google 您认为哪个网址是规范网址](https://support.google.com/webmasters/answer/139066)，但 Google 仍可能会因各种原因而选择其他网址作为规范网址。

下面简要说明了这些术语，以及这些术语在 Search Console 中的用法：

* **文档**：一个类似网页的集合。包含规范网址，如果您的网站有重复网页，还包含备用网址。文档中的网址可能来自相同或不同的组织（根域名，例如 www.google.com 中的“google”）。Google 会根据平台（移动设备/桌面设备）、用户语言‡或地理位置以及多个其他变量，选择要显示在搜索结果中的最佳网址。Google 可通过自然抓取或网站实现的功能发现网站上的相关网页，这些功能包括重定向或 `<link rel=alternate/canonical>` 标记。其他组织的相关网页只有在您网站通过重定向或链接标记明确编码的情况下才会被标记为备用网页。
* **网址**：用于访问网站上指定内容的网址。网站可能会将不同网址解析为指向同一网页。
* **网页**：通过一个或多个网址访问的指定网页。网页可能有不同的版本，具体取决于用户的平台（移动设备、桌面设备、平板电脑等）。
* **版本**：网页的一个变体，通常分为“移动版”、“桌面版”和“AMP”（但 AMP 网页本身可以有移动版和桌面版）。每个版本都可以有不同网址（example.com 与 m.example.com）或相同网址（如果您的网站[动态提供内容](https://developers.google.com/search/mobile-sites/mobile-seo/dynamic-serving)或使用[自适应设计](https://developers.google.com/search/mobile-sites/mobile-seo/responsive-design)，那么同一网址可以显示同一网页的不同版本），具体取决于您的网站配置。语言变体不会被视为不同版本，而是被视为不同的文档。
* **规范网页或网址**：Google 认为最能代表文档的网址。Google 始终会抓取此网址，偶尔也会抓取文档中的重复网址。
* **备用/重复网页或网址**：Google 可能会偶尔抓取的文档网址。如果这些网址适合用户和请求，Google 也会呈现这些网址（例如，会为在桌面设备上提出请求的桌面设备用户提供备用网址，而不是规范移动网址）。
* **网站 \(Site\)**：通常用作网站 \(website\)（概念相关的一组网页）的同义词，但有时也可用作 Search Console 资源的同义词，而实际上可以将资源定义为网站的一部分。网站可以跨子网域（甚至跨组织，如果具有正确关联的 AMP 网页的话）。

**‡**采用不同语言但具有相同内容的网页会存储在不同文档中，这些文档使用 [hreflang 标记](https://support.google.com/webmasters/answer/189077)相互引用；这就是为什么务必要用 hreflang 标记翻译内容的原因。

#### 呈现结果 <a id="serving_advanced"></a>

用户输入查询时，我们的机器会在索引中搜索匹配网页，并返回我们认为与用户搜索最相关的结果。相关性是由数百个因素决定的，我们一直在努力改进算法。Google 在选择结果和对其进行排名时会考虑用户体验，因此请务必确保您的网页能[快速加载](https://developers.google.com/speed/)且[适合移动设备](https://developers.google.com/search/mobile-sites/)。

**改善结果呈现**

* 如果您的结果针对的是特定地点或使用特定语言的用户，可以[告知 Google 您的偏好](https://support.google.com/webmasters/answer/182192)。
* 确保您的网页能[快速加载](https://developers.google.com/speed/)且[适合移动设备](https://developers.google.com/search/mobile-sites/)。
* 遵循[网站站长指南](https://support.google.com/webmasters/answer/answer.py?answer=35769)，避免常见的潜在问题并提高网站排名。
* 考虑为您的网站[实施搜索结果功能](https://developers.google.com/search/docs/guides/search-gallery)，例如食谱卡片或文章卡片。
* [实施 AMP](https://developers.google.com/search/docs/guides/about-amp)，以加快网页在移动设备上的加载速度。某些 AMP 网页也可以使用其他搜索功能，例如“焦点新闻”轮换展示。
* Google 的算法一直在不断改进，您应遵循我们的指南，努力创建符合用户需求的精彩内容，而不应尝试去猜测算法并根据算法来设计网页。

### 更详尽的版本

[点击此处即可找到对 Google 搜索工作方式更详尽的介绍](https://www.google.com/search/howsearchworks/)（含有图片和视频！）

