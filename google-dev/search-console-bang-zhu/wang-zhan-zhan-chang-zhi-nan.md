# 网站站长指南

## 网站站长指南

遵循以下**常规指南**有助于 Google 找到您的网站、将其编入索引并对其进行排名。

我们强烈建议您多加留意以下**质量指南**，其中简要说明了一些违规行为，这些行为可能会导致网站被从 Google 索引中彻底移除，或者导致系统算法或我们手动将其标识为垃圾网站。如果某个网站被标识为垃圾网站，那么它可能不会再出现在 Google.com 或任何 Google 合作伙伴网站的搜索结果中。

### 常规指南

#### 帮助 Google 找到您的网页帮助 Google 了解您的网页帮助访问者使用您的网页

* 确保网站上的所有网页均能通过其他可找到的网页上的链接转到。引荐链接应包括与目标网页相关的文字（对于图片，则应包括 Alt 属性）。可供抓取的链接是[含有 href 属性的 `<a>` 标记](https://support.google.com/webmasters/answer/9112205)。
* 提供 [Sitemap 文件](http://sitemaps.org/)，其中应含有指向您网站上的重要网页的链接。同时，也请提供一个列出了所有指向这些网页的用户可读链接的页面（也称为网站索引或站点地图页面）。
* 将网页上的链接数量限制在合理范围内（最多几千个）。
* 确保您的网络服务器能够正确地支持 `If-Modified-Since` HTTP 标头。此功能可指示您的网络服务器告诉 Google，自我们上次抓取您的网站以来，您的内容是否发生了变化。支持此功能可以节省带宽和开销。
* 在您的网络服务器上使用 robots.txt 文件，通过防止抓取无限的区域（例如搜索结果页）来管理您的抓取预算。及时更新您的 robots.txt 文件。[了解如何借助 robots.txt 文件来管理抓取流程](https://developers.google.com/webmasters/control-crawl-index/docs/faq)。使用 [robots.txt 测试工具](https://www.google.com/webmasters/tools/robots-testing-tool)测试 robots.txt 文件的覆盖率和语法。

   **如何帮助 Google 找到您的网站：**

* [请求 Google 抓取您的网页](https://support.google.com/webmasters/answer/6065812)。
* 对于应知道您网页情况的所有网站，请务必通知它们您的网站已上线。

#### 帮助 Google 了解您的网页

* 创建一个实用且信息丰富的网站，并撰写能够清晰准确地表述内容的网页。
* 要考虑到用户会使用哪些字词来查找您的网页，并确保您的网站上确实包含了这些字词。
* 确保您的 `<title>` 元素和 `alt` 属性均为描述性内容且具体、准确。
* 网站的页面层次结构要具有一个清晰的概念。
* 遵循我们建议的[图片](https://support.google.com/webmasters/answer/114016)、[视频](https://support.google.com/webmasters/answer/156442)和[结构化数据](https://developers.google.com/structured-data/)最佳做法。
* 使用内容管理系统（例如 Wix 或 WordPress）时，确保该系统生成的网页和链接能被搜索引擎抓取。
* 为了使 Google 能全面了解网站的内容，请允许 Google 抓取可能会显著影响网页呈现效果的所有网站资源，例如会影响 Google 了解网页的 CSS 和 JavaScript 文件。Google 索引系统所呈现的网页样貌与用户看到的一样，包括图片、CSS 和 JavaScript 文件。要查看 Googlebot 无法抓取的网页资源，请使用[网址检查工具](https://support.google.com/webmasters/answer/9012289)；要调试 robots.txt 文件中的指令，请使用 [robots.txt 测试](https://support.google.com/webmasters/answer/6062598)工具。
* 允许搜索漫游器在不使用可跟踪其网站访问路径的会话 ID 或网址参数的情况下抓取您的网站。这些技术有助于跟踪个人用户行为，但漫游器的访问模式完全不同。采用这些技术可能会导致对网站的索引编制不完整，因为漫游器可能无法排除那些看似不同但实际却指向同一个网页的网址。
* 确保在默认情况下 Google 能够查看您网站的重要内容。Google 能够抓取隐藏在导航元素（例如标签或展开部分）中的 HTML 内容，但我们认为用户不太容易访问此类内容，因此您应在默认页视图中显示最重要的信息。
* 尽可能确保网页上的广告链接不会影响搜索引擎排名。例如，使用 [robots.txt](https://support.google.com/webmasters/answer/156449)、`rel="nofollow"` 或 `rel="sponsored"` 阻止抓取工具跟踪广告链接。

#### 帮助访问者使用您的网页

* 尽量使用文字（而非图片）显示重要的名称、内容或链接。如果不得不使用图片代替文字内容，可以使用 `alt` 属性添加一些描述性文字。
* 确保所有链接都能转到实际的网页。使用[有效的 HTML](https://validator.w3.org/)。
* 优化网页加载时间。飞快的网站加载速度既可令用户满意，也可改善网页的整体质量（尤其是对于互联网连接速度较慢的用户而言）。Google 建议您使用 [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) 和 [Webpagetest.org](http://www.webpagetest.org/) 等工具测试网页的加载速度。
* 针对所有设备类型和尺寸（包括桌面设备、平板电脑和智能手机）设计您的网站。使用[移动设备适合性测试工具](https://search.google.com/test/mobile-friendly)测试您的网页在移动设备上的运行效果，并获取反馈以了解哪些内容需要得到修正。
* 确保您的网站[在不同的浏览器中都能正常显示](https://support.google.com/webmasters/answer/100782)。
* 如果可以的话，使用 HTTPS [确保网站的连接安全无虞](https://support.google.com/webmasters/answer/6073543)。对用户与网站之间的互动操作进行加密是在网络上进行通信的最佳做法。
* 确保有视觉障碍的读者也可以浏览您的网页，例如，通过屏幕阅读器测试易用性。

### 质量指南 <a id="quality_guidelines"></a>

这些质量指南涵盖了最常见的作弊形式或操纵行为，对于此处未列出的其他误导行为，Google 也会进行查处。切勿抱有侥幸心理，认为某种欺骗手段未在本页中列出，Google 就会认可该手段。作为网站站长，与其花费大量时间寻找漏洞加以利用，不如尽力维护基本原则，以便为用户带来更好的体验，从而使网站获得更高的排名。

如果您认为其他网站在滥用 Google 的质量指南，请通过垃圾信息举报告知我们。Google 希望能开发出灵活的自动解决方案来解决上述问题，并会使用该报告进一步改进我们的网络垃圾检测系统。[Google 会对垃圾网页执行手动操作吗？](https://www.youtube.com/watch?v=2oPj5_9WxpA)

{% embed url="https://youtu.be/2oPj5\_9WxpA" caption="Matt Cutts 介绍 Google 会对垃圾网页执行的手动操作" %}

#### 基本原则

* 您在设计网页时主要考虑的应该是用户，而不是搜索引擎。
* 不要欺骗用户。
* 不要为了提高搜索引擎排名而弄虚作假。一种可以有效分辨您的做法是否妥当的衡量标准是，您在向竞争对手网站或 Google 员工解释自己的作为时是否感到坦然。另一个有用的测试手段是扪心自问：“这能否给我的用户带来帮助？如果没有搜索引擎，我会这样做吗？”
* 想一想是什么让您的网站变得独一无二、具有价值或吸引力。让您的网站在相应领域中出类拔萃。

#### 具体指南

**避免**使用以下技术：

* [自动生成的内容](https://support.google.com/webmasters/answer/2721306)
* 参与[链接方案](https://support.google.com/webmasters/answer/66356)
* 制作[原创内容很少或没有原创内容的网页](https://support.google.com/webmasters/answer/66361)
* [隐藏真实内容](https://support.google.com/webmasters/answer/66355)
* [欺骗性重定向](https://support.google.com/webmasters/answer/2721217)
* [隐藏文字或链接](https://support.google.com/webmasters/answer/66353)
* [门页](https://support.google.com/webmasters/answer/2721311)
* [抄袭内容](https://support.google.com/webmasters/answer/2721312)
* 参与[不能有效增值的联属计划](https://support.google.com/webmasters/answer/76465)
* 加载带有[无关关键字](https://support.google.com/webmasters/answer/66358)的网页
* 制作会实施[恶意行为](https://support.google.com/webmasters/answer/2721313)（如网上诱骗，或安装病毒、特洛伊木马或其他有害软件）的网页
* 滥用[结构化数据](https://developers.google.com/search/docs/guides/sd-policies)标记
* 向 Google 发送[自动查询](https://support.google.com/webmasters/answer/66357)

**遵守**最佳做法，例如：

* 监控网站上的[黑客攻击](https://support.google.com/webmasters/answer/2721435)活动并尽快移除被黑的内容
* 防止网站上出现[用户生成的垃圾内容](https://support.google.com/webmasters/answer/2721437)并及时移除

如果您的网站违反以上一条或多条指南，则 Google 可能会对您的网站执行[手动操作](https://support.google.com/webmasters/answer/9044175)。一旦解决了这些问题，您便可提交网站以供[重新审核](https://support.google.com/webmasters/answer/35843)。  


