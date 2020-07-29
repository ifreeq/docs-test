---
description: 让 Google 搜索能够发现您的视频
---

# 视频最佳做法

人们每天通过 Google 执行数十亿次搜索，其中有许多次是在查找视频内容。如果您遵循下面列出的最佳做法（以及我们的常规[网站站长指南](https://support.google.com/webmasters/answer/35769)），就可以提高您的视频出现在搜索结果中的几率。

Google 搜索中的视频结果会同时显示在合并搜索结果和视频搜索结果中。当用户点击视频结果时，系统会将他们转到您的网页（用户可在此观看您的视频）。

* [Google 如何抓取视频](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#how-google-crawls)
* [关于视频搜索结果](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#video-search-results)
* [最佳做法](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#best-practices)
* [如何区分各个网址](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#which-url-is-which)
* [从 Google 搜索结果中屏蔽特定视频](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#block_video)
* [常见的视频索引编制错误](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#common_errors)

{% embed url="https://youtu.be/P1YrtGkwm5M" %}

### Google 如何抓取视频

要使视频出现在搜索结果中，Google 必须了解该视频。Google 可以通过以下方式提取视频的相关信息：

* Google 可以抓取该视频（如果采用[支持的视频编码](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#file-types)）并提取缩略图和预览信息。Google 也可以从相应文件的音频和视频中提取一些有限的含义。
* Google 可以从托管该视频的网页中提取信息，包括网页文本和元标记。
* Google 可以使用与该视频关联的结构化数据 \([VideoObject](https://developers.google.com/search/docs/data-types/video)\) 或[视频 Sitemap](https://support.google.com/webmasters/answer/80471)。

**YouTube 内容**：YouTube 视频始终都可供抓取。不过，提供视频 Sitemap 或结构化数据仍然很有帮助，此类内容可帮助 Google 在您的网页上找到嵌入式 YouTube 视频。此外，站点地图和结构化数据还有助于您向我们提供有关视频的其他信息。

### 关于视频搜索结果

您的视频如何（或能否）显示在搜索结果中取决于您提供给 Google 的信息量。Google 需要两项信息才能将您的视频显示在搜索结果中：缩略图和指向实际视频文件的链接。但是，您提供的信息越多，搜索结果体验就越好。

以下是视频搜索结果呈现的两个基本级别：

* **基本搜索结果呈现**：只要您向 Google 提供最低限度的信息，您的视频就能够以缩略图和链接的形式显示在合并搜索结果和视频搜索结果中。但是您不会获得任何增强功能，例如视频预览或内容解析。最低限度的信息是缩略图和指向视频文件的链接。

  ![](https://lh3.googleusercontent.com/AM3p4OOlOZve_luCcU6EJzIijzE22n--S13A6VA5u9sHkN2VHhuLZsoTR1dGu-_2rI06=w500)  
  基本视频搜索结果示例

* **增强型搜索结果呈现**：如果您提供更多信息，Google 就可针对您的视频提供更多功能，例如视频预览、视频时长、视频日期和提供商信息、根据用户所在国家/地区或所用搜索设备限制搜索结果，等等。

  ![Sample desktop video search result](https://lh3.googleusercontent.com/VBWrDcJ-66dezin7HYNeyp1fx_iD9tQsbLTdSto5aitH6M2OkBvg-JzcwA22TIooQCw=w250)  
  增强型桌面视频搜索结果示例

  ![Sample mobile video search result](https://lh3.googleusercontent.com/k1FUSMatP4-D84YVwE1nZfdUOztfTXiuzIBWYbFSWJSHBUea7y3ZYysT6SNu0ddGC-o=w300)  
  增强型移动视频搜索结果示例

### 最佳做法

**视频搜索结果的最低要求：**

如果您希望您的视频能够显示在搜索结果中，必须满足以下要求：

* **Google 必须能够找到该视频**。系统会根据是否存在某种 HTML 标记（例如 `<video>`、`<embed>` 或 `<object>`）来识别网页中的视频。请确保网页不[需要复杂的用户操作或特定的网址片段即可加载](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#complex-javascript)，否则 Google 可能找不到它。**提示**：虽然我们可以通过自然抓取找到网页中内嵌的视频，但您也可以通过发布[视频 Sitemap](https://support.google.com/webmasters/answer/80471) 帮助我们找到您的视频。
* **您必须**[**为视频提供高品质的缩略图**](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#thumbnails)**。**
* **确保每个视频都位于可公开访问的网页中**，用户可以在其中观看视频。该网页不应该要求用户登录，也不应该被 [robots.txt](https://support.google.com/webmasters/answer/6062608) 或 [noindex](https://support.google.com/webmasters/answer/93710) 屏蔽，必须可供 Google 访问。
* **视频内容应确切吻合其所在网页的内容**。例如，如果您拥有一个介绍桃饼的食谱网页，不要嵌入笼统介绍甜点的视频。
* 确保您在视频 Sitemap 或视频标记中提供的任何信息与实际视频内容**一致**。

  
**为了达到最佳效果，请注意以下事项：**

如果您执行这些额外的步骤，Google 就可以为您的视频提供更好的搜索结果：

* [确保 Google 可以抓取您的视频文件](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#make-crawlable)。如果 Google 可以抓取该文件，便可以为您生成缩略图并提供其他功能。
* [避免使用复杂的逻辑或脚本来显示或隐藏视频](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#loading_conditionals)。如果 Google 需要完成复杂的用户互动或满足复杂的资源加载条件，Google 可能无法找到您的所有视频。
* [在视频托管网页中提供良好的用户体验](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#good-experience)。
* 提供描述视频的[结构化数据](https://developers.google.com/search/docs/data-types/video)和/或[视频 Sitemap](https://support.google.com/webmasters/answer/80471)。如果 Google 已经知悉该网页，结构化数据是最佳选择；站点地图可以帮助 Google 找到新内容或了解新内容或更改过的内容。有些网站会针对所有视频使用页内结构化数据，同时使用视频 Sitemap 将新视频、限时视频或难以找到的视频告知 Google。
  * **测试结构化数据或站点地图**：
    * 您可以使用[富媒体搜索结果测试工具](https://search.google.com/test/rich-results)测试视频结构化数据。
    * 如需进行测试，或需要提交站点地图或 Feed，请先在 Search Console 中[添加并验证](https://support.google.com/webmasters/answer/34592)您的网站。确保您已验证包含站点地图或 Feed 的网站，以及该站点地图或 Feed 中引用的所有网站。在 [Search Console 站点地图工具](https://search.google.com/search-console/sitemaps)中测试或提交站点地图。
* 在结构化数据或站点地图中为每个视频提供独特的描述性数据。
* 将[任何内容更新](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#updating-content)告知 Google。
* [正确地处理已移除的视频](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#remove_video)。
* [在 Google Search Console 中添加并验证您的网站](https://support.google.com/webmasters/answer/34592)。Search Console 会显示 Google 在抓取您的网站时遇到的所有问题。如果您使用了站点地图，请[通过 Search Console 提交站点地图，或将其列在 robots.txt 文件中](https://support.google.com/webmasters/answer/183668)。
* 了解[常见的视频索引编制错误](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#common_errors)。
* 如果您有直播视频，您可以在搜索结果中[将其标记为直播](https://developers.google.com/search/docs/data-types/livestream)。

#### 为视频提供高品质缩略图

视频必须具有可显示在搜索结果中的缩略图，才能出现在 Google 视频搜索结果中。

**您可以通过以下几种方式提供（或启用）缩略图：**

* 如果使用 `<video>` HTML 标记，请指定 `poster` 属性。
* 在视频 Sitemap 中，指定 `<video:thumbnail_loc>`
* 在结构化数据中，指定 `VideoObject.thumbnailUrl`
* 以[可抓取的格式](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#make-crawlable)提供视频，我们可以为您生成缩略图。

**首选格式**：JPG、PNG

**尺寸**：介于 160x90 和 1920x1080 像素之间

**位置**：预览缩略图必须可供 Googlebot 访问，即未被 [robots.txt](https://support.google.com/webmasters/answer/6062608) 屏蔽，也不要求登录。

#### 确保您的视频可供抓取

只要 Google 可以抓取您的视频，我们便可以为您生成缩略图、启用视频预览并提供其他功能。

**为确保视频可供抓取，请注意以下事项：**

* 视频必须[采用支持的格式](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#file-types)。
* 不能禁止 Google 访问视频托管网页和实际视频文件。（“已屏蔽”是指该网页或文件受付费墙保护、要求登录、被 [noindex](https://support.google.com/webmasters/answer/93710) 或 [robots.txt](https://support.google.com/webmasters/answer/6062608) 屏蔽。）
* 视频托管网页和流式传输实际视频的服务器必须具有一定的带宽才能被抓取。因此，如果您的着陆页（位于 `example.com/puppies.html`）具有通过 `somestreamingservice.com` 提供的嵌入式小狗视频，则必须取消屏蔽 `example.com` 和 `somestreamingservice.com`，并且二者必须具有足够的服务器负载。

**支持的视频编码**

Google 可抓取以下类型的视频文件：.3g2、.3gp2、.3gp、.3gpp、.asf、.avi、.divx、.f4v、.flv、.m2v、.m3u8、.m4v、.mkv、.mov、.mp4、.mpe、.mpeg、.mpg、.ogv、.qvt、.ram、.rm、.vob、.webm、.wmv、.xap大多数移动网络浏览器已不再支持 Flash，并且 Adobe 计划在 2020 年弃用 Flash。如果您的视频采用 Flash 格式，不妨考虑将其转码为移动浏览器支持的其他格式。

#### 使用结构化数据或视频 Sitemap 描述您的视频

您可以使用结构化数据和/或视频 Sitemap 向 Google 提供有关您视频的其他信息。提供这些额外的信息可以在搜索结果中启用更多功能，并帮助我们更好地了解您的视频和对其进行排名。

这两种技术可以向 Google 呈现相同的信息，但视频 Sitemap 可以帮助 Google 更快地找到新内容或更新内容；相对于站点地图，某些人可能更熟悉结构化数据，并且可能与其网站使用结构化数据实现富媒体搜索结果的方式更加一致。您可以在自己的网站上同时采用这两种技术，但如果您这样做，请确保在使用这两种技术时您的数据是一致的。

**结构化数据**

您可以在托管网页上添加描述视频的结构化数据。结构化数据是指使用标记或 JSON 以明确定义的格式提供的信息。当 Google 抓取此网页时，它可以解读该格式以提取视频的相关信息。

您可以使用多种格式，但 Google 强烈建议使用 schema.org 的 `VideoObject` 语法并采用 JSON-LD 格式。Schema.org VideoObject（**推荐**）

在网页上嵌入 `VideoObject` 代码。`VideoObject` 与包含匹配源网址的嵌入式视频相关联。

[了解如何在网页中为每个视频嵌入 VideoObject 说明](https://developers.google.com/search/docs/data-types/video)。

**VideoObject JSON-LD 示例**

```text
<html>
<head>
  <title>一小时内炸好肉排</title>
</head>
<body>
  <script type="application/ld+json">
  {
    "@context": "http://schema.org",
    "@type": "VideoObject",
    "name": "炸肉排攻略",
    "description": "如何在一小时内就能炸好美味的肉排",
    "thumbnailUrl": "https://example.com/imgs/schnitzel-small.jpg",
    "uploadDate": "2015-02-05T08:00:00+08:00",
    "duration": "PT1M33S",
    "contentUrl": "https://streamserver.example.com/schnitzel.mp4"
  }
  </script>
  <h1>人人都爱炸肉排</h1>

  ... 省略了与炸肉排相关的网页内容...

  <video width="420"
      src="https://streamserver.example.com/schnitzel.mp4"
      poster="https://example.com/imgs/schnitzel-small.jpg"/>
</body>
</html>
```

**使用简单的 VideoObject 还是电视/影片富媒体搜索结果？**

如果您只是想利用评论或演员表等信息描述电视节目或影片，或如果您的视频要求执行购买或租借等复杂的操作，您应在网站上实现[电视或影片结构化数据类型](https://developers.google.com/search/docs/data-types/tv-movie)。使用电视或影片结构化数据可以在 [Google 搜索中显示富媒体搜索结果](https://developers.google.com/search/docs/guides/images/search-gallery-tvMovies01.png)，其中包括评分、评论和演员信息，以及指向免费或付费在线播放服务的链接。富媒体搜索结果只会显示在合并搜索结果窗格中。 Open Graph 协议

**视频 Sitemap**

视频 Sitemap 是 Google 用于在您的网站上查找视频的 XML 站点地图，也可以向 Google 提供视频的相关信息。视频 Sitemap 条目可以使用与 `VideoObject` 结构化数据元素相同的方式描述视频。使用视频 Sitemap 的优势在于，它还可以帮助 Google 找到新视频或更新的视频，并且可以在一个文件中描述多个视频，而无需 Google 抓取每个网页并逐一发现各项更改。

[了解如何创建视频 Sitemap](https://support.google.com/webmasters/answer/80471)。

#### 更新内容

{% embed url="https://youtu.be/l0xW6tmbTI8" %}

您可以在视频发生更改时通知 Google，具体取决于您如何帮助我们查找或读取您的内容。如果您只是替换了视频网址或源文件但未做其他更改，Google 可能不会注意到相应更改。

* **结构化数据**：当您的页内视频结构化数据发生更改时，Google 会在下次抓取该网页时看到相应更改。您可以使用普通站点地图或视频 Sitemap 告知 Google 发生更改的网页。
* **视频 Sitemap 和 mRSS**：当您发布视频 Sitemap 后，Google 会定期重新抓取它，并使用任何已更改的视频数据更新搜索结果。您也可以重新提交站点地图或告知 Google 发生更改的站点地图，以请求立即重新抓取。详细了解如何[提交站点地图并使用 HTTP 请求更新站点地图](https://support.google.com/webmasters/answer/6065812#sitemap)。

#### 移除视频

建议采用以下方式将视频从网站中移除：

* 对于包含已移除或已失效视频的着陆页，**返回 404（未找到）**HTTP 状态代码。除了 404 响应代码之外，您仍可返回网页的 HTML，以便让大多数用户了解实际变化。
* 在 schema.org 结构化数据、视频 Sitemap（使用 `<video:expiration_date>` 元素）或 mRSS Feed（`<dcterms:valid>` 标记）中**指明失效日期**。下面这个视频 Sitemap 示例包含一个已于 2009 年 11 月失效的视频：

  ```text
  <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
          xmlns:video="http://www.google.com/schemas/sitemap-video/1.1"> 
    <url> 
      <loc>http://www.example.com/videos/some_video_landing_page.html</loc>
      <video:video>
        <video:thumbnail_loc>
            http://www.example.com/thumbs/123.jpg
        </video:thumbnail_loc> 
        <video:title>
            适合夏季的烧烤排餐
        </video:title>
        <video:description>
            小鲍教您如何每次都能烤出美味牛排
        </video:description>
        <video:player_loc>
            http://www.example.com/videoplayer?video=123
        </video:player_loc>
        <video:expiration_date>2009-11-05T19:20:30+08:00</video:expiration_date>
      </video:video> 
    </url> 
  </urlset>
  ```

**如果您需要立即从搜索结果中移除视频**，还应[提交移除请求](https://www.google.com/webmasters/tools/removals)。请注意，为了成功完成此请求，该视频不能存在，也不能可供 Google 访问，即返回 404 代码或需要登录。

#### 避免使用复杂的视频加载条件

在设计您的网站时，请合理地配置您的视频网页，以便无需进行过于复杂的用户互动或满足过于复杂的条件就能加载视频。例如，如果您只是在某些情况下使用过于复杂的 JavaScript 在 JavaScript 中嵌入视频对象（例如，在网址中使用哈希标记），那么我们也可能只能找到您的部分视频。如果您未使用站点地图列出视频，这一点尤为重要。

#### 在视频网页中营造精彩的用户体验

除了精彩的视频外，您还应该考虑如何围绕内容设计 HTML 网页。例如，应该考虑以下事项：

* **为每个视频创建独立的着陆页**，您可以在其中收集视频的所有相关信息。如果要执行此操作，请确保在每个网页上提供唯一的信息，例如描述性标题和说明。
* 让用户尽可能方便地在每个着陆页上**查找和播放视频**。在显眼的位置嵌入视频播放器（采用广受支持的视频格式），可以增加视频对用户的吸引力，并且更方便 Google 将其编入索引。

#### 按平台限制用户

您可以根据搜索用户的平台限制视频的搜索结果。平台包括计算机浏览器、移动设备浏览器和电视浏览器。

{% embed url="https://youtu.be/6xdSwPE7sd0" %}

### 使用视频 Sitemap 按平台进行限制

如果您的视频没有任何平台限制，请勿添加平台限制标记。

您可以在视频 Sitemap 中使用 [`<video:platform>` 标记](https://support.google.com/webmasters/answer/80471#platform)允许或阻止视频显示在指定设备上的搜索结果中。每个视频条目只能有 1 个 `<video:platform>` 标记。该标记具有一个必需属性 `relationship`，可指定列出的平台是被排除在外还是必需平台。

**示例**

在以下视频 Sitemap 示例中，视频将仅在桌面设备和移动设备的浏览器上显示。

```text
<url>
  <loc>http://www.example.com/videos/some_video_landing_page.html</loc>
  <video:video>
    <video:thumbnail_loc>
        http://www.example.com/thumbs/123.jpg
    </video:thumbnail_loc>
    <video:title>适合夏季的烧烤排餐</video:title>
    <video:description>
        小鲍教您如何每次都能烤出美味牛排
    </video:description>
    <video:player_loc>
        http://www.example.com/videoplayer?video=123
    </video:player_loc>
    <video:platform relationship="allow">web mobile</video:platform>
  </video:video>
</url>
```

### 使用结构化数据或 mRSS 按平台进行限制

VideoObject 或 mRSS Feed 没有任何平台限制标记。

#### 按国家/地区限制用户

您可以根据搜索用户的地理位置限制视频的搜索结果。如果您的视频没有任何国家/地区限制，请勿添加国家/地区限制标记。

{% embed url="https://youtu.be/ndynLJW6CBQ" %}

### 使用视频 Sitemap 按国家/地区进行限制

您可以在视频 Sitemap 中使用 [`<video:restriction>` 标记](https://support.google.com/webmasters/answer/80471#country-restriction)允许或拒绝视频在特定国家/地区显示。每个视频条目只能有 1 个 `<video:restriction>` 标记。

`<video:restriction>` 标记应该包含一个或多个用空格分隔的 [ISO 3166](http://en.wikipedia.org/wiki/ISO_3166) 国家/地区代码。`relationship` 属性是必需项，可指定限制的类型。

* **`relationship="allow"`** - 视频将仅在指定的国家/地区显示。如果未指定国家/地区，视频不会在任何地方显示。
* **`relationship="deny"`** - 视频将在除指定国家/地区外的所有其他地方显示。如果未指定国家/地区，视频将在所有地方显示。

在以下视频 Sitemap 示例中，视频将仅在加拿大和墨西哥显示。

```text
   <url> 
     <loc>http://www.example.com/videos/some_video_landing_page.html</loc>
     <video:video>
       <video:thumbnail_loc>
           http://www.example.com/thumbs/123.jpg
       </video:thumbnail_loc> 
       <video:title>适合夏季的烧烤排餐</video:title>
       <video:description>
           小鲍教您如何每次都能烤出美味牛排
       </video:description>
       <video:player_loc>
           http://www.example.com/player?video=123
       </video:player_loc>
       <video:restriction relationship="allow">ca mx</video:restriction> 
     </video:video> 
   </url>
```

### 使用结构化数据按国家/地区进行限制

 如果您使用 `VideoObject` 结构化数据描述视频，请设置 [`VideoObject.regionsAllowed` 属性](https://schema.org/VideoObject)，指定哪些区域内的用户可获得视频搜索结果。如果不添加此属性，所有区域内的用户都可在搜索结果中看到该视频。

### 使用 mRSS 按国家/地区进行限制

在 mRSS Feed 中，您可以将 [`media:restriction` 标记](https://support.google.com/webmasters/answer/80471#mrss)的必需属性 `type` 设置为 `country`，为视频指定国家/地区限制。`media:restriction` 还要求将 `relationship` 属性设置为 `allow` 或 `deny`，并接受一系列用空格分隔的 ISO 3166 国家/地区代码。

在以下 mRSS 条目示例中，视频将在除美国和加拿大以外的所有其他地方显示。

```text
  <item xmlns:media="http://search.yahoo.com/mrss/" xmlns:dcterms="http://purl.org/dc/terms/">
    <link>http://www.example.com/examples/mrss/example.html</link>
    <media:content url="http://www.example.com/examples/mrss/example.mp4"
                   fileSize="405321" type="video/x-flv" height="240"
                   width="320" duration="120" medium="video"
                   isDefault="true">
      <media:title>适合夏季的烧烤排餐</media:title>
      <media:description>
          每次都能烤出美味牛排
      </media:description>
      <media:thumbnail
          url="http://www.example.com/examples/mrss/example.png"
          height="120" width="160"/>
    </media:content>
    <media:restriction relationship="deny" type="country">us ca</media:restriction>
  </item>
```

详细了解如何针对 Google 视频搜索[使用 mRSS Feed](https://support.google.com/webmasters/answer/80471#mrss)，或详细了解 [mRSS 规范](http://www.rssboard.org/media-rss#media-restriction)中的 `media:restriction` 标记。

### 如何区分各个网址？

网页上可能有多个网址与某个视频文件相关联。下面简要说明了其中的大多数网址：

![Diagram of URLs in a page](https://lh3.googleusercontent.com/cOrYUJXSEsqESYRI9M9YXDEb0-lcuV3pYEsUkxzMWLJBPGNlDKWQ6ZourcpCC4Ea5lc=w430)

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left">&#x6807;&#x8BB0;</th>
      <th style="text-align:left">&#x8BF4;&#x660E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>1</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li><code>&lt;loc&gt;</code>
            <br />&#xFF08;&#x89C6;&#x9891; Sitemap &#x6807;&#x8BB0;&#xFF09;</li>
        </ul>
      </td>
      <td style="text-align:left">
        <p>&#x89C6;&#x9891;&#x6258;&#x7BA1;&#x7F51;&#x9875;&#x7684;&#x7F51;&#x5740;&#x3002;<b>&#x793A;&#x4F8B;</b>&#xFF1A;</p>
        <p><code>&lt;loc&gt;https://example.com/news/worlds-biggest-cat.html&lt;/loc&gt;</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>2</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li><code>VideoObject.embedUrl</code>
            <br />&#xFF08;&#x7ED3;&#x6784;&#x5316;&#x6570;&#x636E;&#xFF09;</li>
          <li><code>&lt;video:player_loc&gt;</code>
            <br />&#xFF08;&#x89C6;&#x9891; Sitemap &#x6807;&#x8BB0;&#xFF09;</li>
          <li><code>&lt;iframe src=&quot;...&quot;&gt;</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">
        <p>&#x81EA;&#x5B9A;&#x4E49;&#x64AD;&#x653E;&#x5668;&#x7684;&#x7F51;&#x5740;&#x3002;&#x901A;&#x5E38;&#x662F;&#x7F51;&#x9875;&#x4E0A;&#x7684; <code>&lt;iframe&gt;</code> &#x6216; <code>&lt;embed&gt;</code> &#x6807;&#x8BB0;&#x7684;
          src &#x503C;&#x3002;<b>&#x793A;&#x4F8B;</b>&#xFF1A;</p>
        <p><code>&lt;video:player_loc&gt;<br />https://archive.example.org/cats/1234&lt;/video:player_loc&gt;</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>3</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li><code>&lt;video src=&quot;...&quot;&gt;</code>
            <br />&#xFF08;HTML &#x6807;&#x8BB0;&#xFF09;</li>
          <li><code>&lt;embed src=&quot;...&quot;&gt;</code>
            <br />&#xFF08;HTML &#x6807;&#x8BB0;&#xFF09;</li>
          <li><code>&lt;video:content_loc&gt;</code>
            <br />&#xFF08;&#x89C6;&#x9891; Sitemap &#x6807;&#x8BB0;&#xFF09;</li>
          <li><code>VideoObject.contentUrl</code>
            <br />&#xFF08;&#x7ED3;&#x6784;&#x5316;&#x6570;&#x636E;&#xFF09;</li>
        </ul>
      </td>
      <td style="text-align:left">
        <p>&#x5B9E;&#x9645;&#x5185;&#x5BB9;&#x6587;&#x4EF6;&#x7684;&#x7F51;&#x5740;&#xFF0C;&#x8FD9;&#x4E9B;&#x6587;&#x4EF6;&#x53EF;&#x4EE5;&#x4F4D;&#x4E8E;&#x672C;&#x5730;&#x7F51;&#x7AD9;&#x4E0A;&#xFF0C;&#x4E5F;&#x53EF;&#x4EE5;&#x4F4D;&#x4E8E;&#x6D41;&#x5F0F;&#x4F20;&#x8F93;&#x670D;&#x52A1;&#x4E0A;&#x3002;<b>&#x793A;&#x4F8B;</b>&#xFF1A;</p>
        <p><code>&lt;video src=&quot;videos.example.com/cats/1234.mp4&quot;&gt;</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

在添加结构化数据、视频 Sitemap 或备用站点地图时，您应该指向嵌入式播放器或实际文件（具体视相应字段而定）。

### 从 Google 搜索结果中屏蔽特定视频

如果您想从 Google 搜索结果中隐藏某个视频，下面这几种方法都行得通：

* 为托管网页及视频文件显示某种形式的登录屏幕。
* 在视频 Sitemap 中为视频添加[国家/地区限制](https://support.google.com/webmasters/answer/156442?hl=zh-Hans&ref_topic=2370565&dark=0#restrict_by_country)，然后指定一个空的“allow”列表： `<video:restriction relationship="allow"></video:restriction>`
* [使用 robots.txt](https://support.google.com/webmasters/answer/6062608) 屏蔽源视频和/或托管网页。如果视频和托管网页是在同一网站上，则既屏蔽源文件网址（contentUrl 地址）也屏蔽托管网页网址。如果视频托管在其他 CDN 上，只需屏蔽托管网页/播放器网页。
* 向针对托管网页和文件（如果文件存在于您的网页上）的请求返回 [noindex HTTP 响应](https://support.google.com/webmasters/answer/93710)。

请注意，上述方法都无法防止其他网页链接到您的视频或网页。

### 常见的视频索引编制错误

下文介绍了我们见过的一些最常见的视频索引编制错误，以及解决这些错误的建议方法，可提高您的视频显示在搜索结果中的几率。您还应该查看我们的[网站站长指南](https://support.google.com/webmasters/answer/35769)。

#### 使用 robots.txt 屏蔽资源

常见的做法是使用 robots.txt 阻止搜索引擎抓取 JavaScript、视频和图片文件。要让 Google 将视频编入索引，我们必须能够看到在结构化数据或站点地图中指定的缩略图、该视频所在的网页、视频本身以及加载该视频所需的任何 JavaScript 或其他资源。确保您的 robots.txt 规则不会屏蔽上述任何与视频相关的资源。

如果您使用的是视频 Sitemap 或 mRSS，请确保 Google 可以访问您提交的任何站点地图或 mRSS Feed。如果这些内容已被 robots.txt 屏蔽，我们将无法读取它们。

[详细了解 robots.txt](https://support.google.com/webmasters/answer/6062608)。

#### 低画质的缩略图

我们接受任意格式的缩略图，但发现使用 .png 和 .jpg 图片效果最佳。图片必须至少为 160x90 像素，且不得超过 1920x1080 像素。

#### 重复的缩略图、标题或说明

对不同视频使用相同的缩略图、标题或说明可能会影响视频索引编制效果，并且可能会令用户感到困惑。确保每个视频的数据都是唯一的。对于系列性内容，一个常见的问题是多个视频具有相同的标题屏幕缩略图。

#### 失效日期设为过去的时间

如果 Google 发现某个视频的失效日期已过，我们便不会将其收录到任何搜索结果中。这包括站点地图、页内结构化数据以及网站头部中的元失效标记中的失效日期。确保每个视频的失效日期都正确无误。虽然此方法对于视频在失效日期之后不再显示的情形非常有用，但您往往会在无意中将可用视频的失效日期设为过去的日期。如果视频不会失效，请不要包含任何失效信息。

#### 列出已删除的视频

从网页中删除嵌入的视频后，某些网站会用 Flash 播放器告知用户视频已不能观看。这样可能会使搜索引擎出现问题，因此建议选用以下措施：

* 对于包含已移除或已失效视频的着陆页，**返回 404（未找到）**HTTP 状态代码。除了 404 响应代码之外，您仍可返回相关网页的 HTML，以便让大多数用户了解实际情况。
* 在提交给 Google 的页内结构化数据、视频 Sitemap（使用 `<video:expiration_date>` 元素）或 mRSS Feed（`<dcterms:valid>` 标记）中**指明失效日期**。

#### 复杂的 JavaScript 和网址片段

在设计网站时，不要使用任何过于复杂的 JavaScript 配置视频网页，这一点很重要。如果您只是在某些情况下使用过于复杂的 JavaScript 在 JavaScript 中嵌入视频对象，那么我们也可能无法正确地将您的视频编入索引。不支持需要“哈希标记”或[片段标识符](https://en.wikipedia.org/wiki/Fragment_identifier)的内容网址或着陆页网址。另外，在网页上使用 Flash 可能会致使该网页无法被高效地编入索引。为了获得最佳效果，请使用纯 HTML 标记（而不是 Flash）显示您的视频标题和说明。

如果您使用的是页内结构化数据，则应该无需运行 Flash 或其他嵌入式播放器即可提供这些结构化数据。

#### 小型视频、隐藏的视频或很难找到的视频

确保您的视频显示在视频网页上并且很容易找到。Google 建议为每个视频使用一个独立的网页，并且每个视频的描述性标题或说明都必须是独一无二的。视频应显示在网页上的显眼位置，不应处于隐藏状态或很难找到。

