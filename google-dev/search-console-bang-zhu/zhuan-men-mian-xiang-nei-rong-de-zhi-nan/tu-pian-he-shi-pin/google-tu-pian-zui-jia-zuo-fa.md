# Google 图片最佳做法

![&#x84DD;&#x8393;&#x5976;&#x6614;](https://lh3.googleusercontent.com/BdQ1ngugnmV6uT960KOX0G9av7YJF4MnoEdSgi5xBINEC4YYuOp3Q7RUj7i4Cg0tRQ=w884)

Google 图片可以帮助用户以直观的方式在网络上发掘信息。得益于[图片说明](https://blog.google/products/search/get-more-useful-information-captions-google-images/)、醒目[标记](https://webmasters.googleblog.com/2017/08/badges-on-image-search-help-users-find.html)和 AMP 结果等新功能，用户可在快速浏览内容时知悉图片的更多相关信息。

为图片添加更多相关信息可使相应的搜索结果变得更加实用，从而给您的网站带来更优质的流量。您可通过确保自己的图片和网站已针对 Google 图片进行优化，使您的内容更容易被发现。发布图片时请遵循我们的指南，以提高您的图片在 Google 图片搜索结果中出现的几率。

**选择停用图片搜索内嵌链接**

选择停用 Google 图片搜索结果中的内嵌链接后，便可阻止完整尺寸的图片显示在 Google 的图片搜索结果页中。

**要选择停用内嵌链接，请按照以下步骤操作：**

1. 当您收到图片请求时，请检查该请求中的 [HTTP 引荐来源网址标头](https://zh.wikipedia.org/wiki/HTTP_referer)。
2. 如果该请求来自某一 Google 网域，请仅回复 HTTP 200 或 HTTP 204（不要附加任何其他内容）。

Google 仍会抓取您的网页并会看到该图片，但会在搜索结果中显示抓取时生成的缩略图。您随时都可选择停用内嵌链接，且无需重新处理网站上的图片。该操作不会被视为隐藏真实图片，也不会引发手动操作。

您还可以[彻底阻止图片显示在搜索结果中](https://support.google.com/webmasters/answer/35308)。

### 打造超凡的用户体验

要想提升您的内容在 Google 图片中的曝光度，请以提供出色的用户体验为原则：**在设计网页时主要考虑用户，而非搜索引擎**。以下是一些建议：

* **提供适当的相关信息**：确保您的视觉内容与其所在网页的主题相关。我们建议您仅在能为网页增添原创价值的情况下展示图片。我们极不赞成在网页中完全使用非原创的图片和文字内容。
* **优化放置位置**：尽可能将图片放置在相关文字附近。必要时，也可考虑将最重要的图片放置在网页顶部附近。
* **勿将重要文字内嵌在图片中**：避免将文字（特别是网页标题和菜单项等重要的文字元素）内嵌在图片中，因为并非所有用户都能访问这类文字（而且网页翻译工具不适用于图片）。为了尽可能让更多的人能访问您的内容，请使用 HTML 格式提供文本，并为图片提供替代文本。
* **创建信息丰富的优质网站**：对 Google 图片而言，优质的网页内容与视觉内容同等重要 - 它可以提供背景信息并更能吸引用户点击搜索结果。网页内容可用于为图片生成一段文本摘要，而且 Google 在进行图片排名时会考虑[对应的网页内容质量](https://webmasters.googleblog.com/2011/05/more-guidance-on-building-high-quality.html)。
* **创建适合在各种设备上访问的网站**：比起桌面设备，用户更多地使用移动设备在 Google 图片上进行搜索。因此，有必要[设计一个适合所有设备类型和尺寸的网站](https://developers.google.com/search/mobile-sites/)。请使用[移动设备适合性测试工具](https://search.google.com/test/mobile-friendly)测试您的网页在移动设备上的运行效果，并获取反馈以了解哪些内容需要修正。
* **为图片创建良好的网址结构**：Google 会借助网址路径以及文件名来理解您的图片。我们建议您好好组织图片内容，以使网址结构合乎逻辑。

### 检查网页标题和说明

| ![](https://lh3.googleusercontent.com/CqaR8po58brlsor-p4N8R41t9F3PyeGIuQ5ZJHNkPGXRqExGdGGPHeCWeirgq_n4YOw7=w720) | ![](https://lh3.googleusercontent.com/fGGVwzupOgWxnmJdU0r48RaDswJwbF42j2e-NGvNjkr2mnix_Mhe7xgjeN12_tPuI_c=w730) |
| :--- | :--- |


Google 图片会自动生成标题和摘要，以充分描述每条结果并表明相应结果与用户查询有何关系。这可帮助用户决定是否要点击某条搜索结果。

我们会使用很多不同的来源生成此信息，例如标题中的描述性信息以及每个网页的元标记。

遵循 [Google 的标题和摘要准则](https://support.google.com/webmasters/answer/35624)就可以帮助我们改善为您的网页显示的标题和摘要的质量。

### 添加结构化数据

如果您添加了结构化数据，Google 图片就能以富媒体搜索结果的形式（包括使用[醒目标记](https://webmasters.googleblog.com/2017/08/badges-on-image-search-help-users-find.html)）展示您的图片，这种做法不仅为用户提供与您的网页相关的信息，还能为您的网站带来更有针对性的流量。Google 图片支持以下类型的结构化数据：

* [产品](https://developers.google.com/search/docs/data-types/product)
* [视频](https://developers.google.com/search/docs/data-types/video)
* [食谱](https://developers.google.com/search/docs/data-types/recipe)

请遵循[结构化数据常规准则](https://developers.google.com/search/docs/guides/sd-policies)以及专门针对您的结构化数据类型的准则，否则您的结构化数据可能无法在 Google 图片中显示为富媒体搜索结果。对于上述每一种结构化数据类型，您都必须填写图片属性字段，才能使数据在 Google 图片中显示为标记和富媒体搜索结果。

| ![](https://lh3.googleusercontent.com/JPEwZ770S_3x6eqPt5v64Gv2o37gZZDjL_yi5QtVvRGuNemc7UxGSecP7SBCCFMOJTA=w373) | ![](https://lh3.googleusercontent.com/zQ3k3Q2toJkMJ8HP5pIY7BlQebsybScbsH6WV86nwdhontpMy-__i3Xe8Pg3mMQFxDM=w726) |
| :--- | :--- |


### 优化网站速度

图片通常是影响整体网页大小的最大因素，可能会导致网页在加载时既速度缓慢又开销巨大。请务必采用[最新的图片优化技术](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization)和[自适应图片技术](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images)，以提供优质且高速的用户体验。

在 Google 图片上，AMP 徽标 ![](https://lh3.googleusercontent.com/piBCOeIarlaZ_L-gfZ1A_LCS7gPIUrQW29732lRrU5cOfHnmbcAmwApRiTCIHCXBxwkH=w36) 有助于用户识别哪些网页能够快速且流畅地加载。您不妨考虑将图片托管网页更改为 [AMP](https://www.ampproject.org/) 网页以缩短网页加载时间（在这种情况下，目标网页即是指用户在 Google 图片中点击搜索结果后到达的网页）。

您可以使用 [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) 分析网站的速度，并访问我们的[“网站开发基础”页面](https://developers.google.com/web/fundamentals/performance/why-performance-matters/)以了解与如何改善网站性能相关的最佳做法和技术。

### 添加高画质图片

与模糊不清的图片相比，高画质图片对用户更有吸引力。另外，搜索结果略缩图中的清晰图片也更能吸引用户，因而可以提高获取用户流量的可能性。

### 为图片添加描述性的标题、说明、文件名和文字

Google 会从图片所在网页的内容中提取与图片主题有关的信息（包括图片说明和图片标题）。请尽可能确保将图片放置在与其相关的文字附近以及与其主题相关的网页上。

同样，Google 也可通过文件名推断图片的主题。例如，**my-new-black-kitten.jpg** 比 **IMG00023.JPG** 更适合作为文件名。

### 使用描述性的替代文本

{% embed url="https://youtu.be/3NbuDpB\_BTc" %}

替代文本是一段描述图片的文本，可协助看不到网页中图片的用户（包括使用屏幕阅读器或网络连接带宽偏低的用户）了解图片内容。

Google 会结合使用替代文字与计算机视觉算法和页面内容来理解图片的主题。如果您决定将图片用作链接，图片的替代文本还可作为定位文字。

在选择替代文本时，请创建实用、信息丰富的内容，并且内容要恰当使用关键字且与网页内容相符。请避免在 Alt 属性中滥用关键字（[关键字堆砌](https://support.google.com/webmasters/answer/66358)），因为这会导致不良的用户体验，并且可能会导致您的网站被视为垃圾网站。

* **效果欠佳（缺少替代文本）：**`<img src="puppy.jpg"/>`
* **效果欠佳（关键字堆砌）：**`<img src="puppy.jpg" alt="puppy dog baby dog pup pups puppies doggies pups litter puppies dog retriever  labrador wolfhound setter pointer puppy jack russell terrier puppies dog food cheap dogfood puppy food"/>`
* **效果较佳：**`<img src="puppy.jpg" alt="puppy"/>`
* **效果最佳：**`<img src="puppy.jpg" alt="Dalmatian puppy playing fetch"/>`

我们建议您通过[审核无障碍性](https://chrome.google.com/webstore/detail/accessibility-developer-t/fpkknkljclfencbdbgkenhalefipecmb)和[使用慢速网络连接模拟器](https://developers.google.com/web/tools/chrome-devtools/network-performance/network-conditions#emulate_network_connectivity)测试您的内容。

### 帮助我们发现您的所有图片

#### 为图片使用语义标记

Google 会解析网页的 HTML 以将图片编入索引，但不会将 CSS 图片编入索引。

* **效果良好：** `<img src="puppy.jpg" alt="A golden retriever puppy" />`
* **效果欠佳：** `<div style="background-image:url(puppy.jpg)">A golden retriever puppy</div>`

#### 使用图片站点地图

图片是网站内容相关信息的重要来源。通过[向图片站点地图添加信息](https://support.google.com/webmasters/answer/178636)，您可以向 Google 提供关于图片的更多详细信息，并提供我们可能无法通过其他方式发现的图片的网址。

与一般的站点地图（设有强制性的跨网域限制）不同，图片站点地图可以包含来自其他网域的网址。这意味着您可以使用 CDN（内容分发网络）托管图片。我们建议您在 Search Console 中验证 CDN 的域名，以便我们能在发现任何抓取错误时通知您。

#### 支持的图片格式

Google 图片支持以下格式的图片：BMP、GIF、JPEG、PNG、WebP 和 SVG。

您还可以将图片内嵌为数据 URI。数据 URI 提供了一种内嵌图片等文件的方式，可以通过将 `img` 元素的 `src` 设置为采用 Base64 编码的字符串来实现。您可以使用以下格式：

```text
<img src="data:image/svg+xml;base64,[data]">
```

虽然内嵌图片能够减少 HTTP 请求，但您应慎重判断何时使用这种图片，因为这种图片可能会导致网页大小大幅增加。要了解详情，请参阅[我们的“网站开发基础”页面中的“内嵌图片的利与弊”部分](https://developers.google.com/web/fundamentals/design-and-ux/responsive/images#inlining_pros_cons)。

#### 自适应图片

设计自适应网页可以带来更好的用户体验，因为用户可以在各自设备上使用这些网页。要了解与如何处理网站中的图片相关的最佳做法，请参阅我们的[“网站开发基础”页面中的“图片”一文](https://developers.google.com/web/fundamentals/design-and-ux/responsive/images)。

网页使用 `<img srcset>` 属性或 &lt;picture&gt; 元素指定自适应图片。但是，某些浏览器和抓取工具无法理解这些属性。我们建议您始终通过 `img src` 属性指定后备网址。

借助“srcset”属性，您可指定同一图片的不同版本，特别是针对不同屏幕尺寸。

**示例：**`<img srcset>`

```text
<img srcset="example-320w.jpg 320w,
             example-480w.jpg 480w,
             example-800w.jpg 800w"
     sizes="(max-width: 320px) 280px,
            (max-width: 480px) 440px,
            800px"
     src="example-800w.jpg" alt="responsive web!">
```

`<picture>` 元素是一个容器，用于对同一图片的不同 &lt;source&gt; 版本进行分组。它提供了一种后备方法，让浏览器能够根据设备功能（例如像素密度和屏幕尺寸）选择合适的图片。对于尚不支持新图片格式的客户端而言，`picture` 元素也非常便于利用内置的优雅降级功能处理新图片格式。

我们建议您在使用 `picture` 标记时，始终提供 img 元素（带 src 属性）作为后备，格式如下：

**示例：**`<picture>`

```text
<picture>
  <source type="image/svg+xml" srcset="pyramid.svg">
  <source type="image/webp" srcset="pyramid.webp"> 
  <img src="pyramid.png" alt="large PNG image...">
</picture>
```

### 针对安全搜索进行优化

安全搜索是帐号中的一项设置，用于指定是要在 Google 搜索结果中显示还是要从中屏蔽包含露骨内容的图片、视频和网站。您应该帮助 Google 了解您图片的性质，以便系统酌情为您的图片应用安全搜索设置。

#### 将仅限成人浏览的图片归到同一个网址位置中

如果您的网站包含成人图片，我们**强烈建议**将这些图片单独归为一组（以与您网站上的其他图片区分开来）。例如：http//www.example.com/adult/image.jpg。

#### 向成人网页添加元数据

当用户开启安全搜索过滤器后，我们的算法会根据各种信号来判断是否应从搜索结果中滤除某张图片或某个完整网页。就图片而言，这些信号中的一部分是通过机器学习技术生成的，但安全搜索算法也会考虑一些更简单的因素，例如图片曾被用于何处以及曾被在什么样的情境中使用过。

最有决定性的信号之一是网页自身是否已被标记为成人网页。如果您想发布成人内容，我们建议您在自己的网页中添加下面的其中一种元标记：

```text
<meta name="rating" content="adult" />
<meta name="rating" content="RTA-5042-1996-1400-1577-RTA" />
```

很多用户都不希望他们的搜索结果中出现成人内容（特别是在孩子也使用同一部设备的情况下）。提供上面的任意一种元标记都有助于提供更佳的用户体验，因为用户不会看到他们不想或不期望看到的结果。

### 最后…

请阅读我们的[搜索引擎优化 \(SEO\) 新手指南](https://support.google.com/webmasters/answer/7451184)，其中提供了许多与如何提升排名相关的实用信息；如果您有其他疑问，请在[网站站长帮助论坛](https://support.google.com/webmasters/community)中发帖咨询。

