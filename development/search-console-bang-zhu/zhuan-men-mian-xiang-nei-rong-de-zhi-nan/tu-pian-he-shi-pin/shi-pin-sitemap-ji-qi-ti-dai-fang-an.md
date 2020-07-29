# 视频 Sitemap 及其替代方案

视频 Sitemap 是一个[站点地图](https://support.google.com/webmasters/answer/156184)，其中包含有关托管在您网页上的视频的其他信息。创建视频 Sitemap 可有效帮助 Google 找到并了解您网站上的所有视频内容，避免漏掉某些内容，尤其是最近添加的内容或 Google 通过常规抓取机制可能发现不了的内容。Google 视频 Sitemap 是对[站点地图标准](http://www.sitemaps.org/)的扩展。

虽然 Google 建议您使用视频 Sitemap，但我们也支持 [mRSS Feed](https://support.google.com/webmasters/answer/80471?hl=zh-Hans&ref_topic=2370565&dark=0#mrss)。您应该遵循[视频最佳做法](https://support.google.com/webmasters/answer/156442)，以便在 Google 搜索中获得最佳效果。

### 视频 Sitemap 准则

**以下是视频 Sitemap 的基本准则：**

* 您既可以单独为视频创建站点地图，也可以在现有站点地图中嵌入视频 Sitemap，哪种方式更方便就选哪种。
* 您可以在一个网页上托管多个视频。
* 每个站点地图条目是托管一个或多个视频的网页的网址。每个站点地图条目的结构如下所示：

  ```text
  <url>
     <loc>https://example.com/mypage</loc>      <!-- 托管网页的网址 -->
     <video> ... 视频 1 的相关信息 ... </video>
     ... 根据需要添加任意多个其他 <video> 条目...
  </url>
  ```

* 不要列出与托管网页不相关的视频。例如，如果视频只是对网页的小小增补，或者与主要文字内容不相关，则不要列出此类视频。
* 视频 Sitemap 中的每个条目都包含您提供的一组必需值、推荐值或可选值。推荐值和可选值可提供实用的元数据，这些元数据能完善您的视频搜索结果并帮助 Google 将您的视频收录到搜索结果中。[要查看站点地图的一系列元素，请参阅下表](https://support.google.com/webmasters/answer/80471?hl=zh-Hans&ref_topic=2370565&dark=0#schema_tags)。
* Google 可能会使用视频着陆页上的文字，而不使用您在站点地图中提供的文字（如果网页文字被视为比站点地图中的信息更为实用的话）。
* 由于 Google 采用复杂的索引编制算法，因此无法保证会将您的视频编入索引，也无法保证何时会将视频编入索引。
* 如果 Google 无法在您提供的网址上发现视频内容，则站点地图条目将被忽略。
* 您提供的每个 Sitemap 文件**所包含的网址元素都不得超过 5 万个**。如果视频超过了 5 万个，您可以提交[多个站点地图](https://support.google.com/webmasters/answer/75712)和[站点地图索引文件](https://support.google.com/webmasters/answer/71453)。您无法嵌套站点地图索引文件。请注意，如果您要添加可选标记，可能您还未达到视频数量上限（5 万个），就已经达到未压缩文件大小的上限 \(50 MB\)。
* Google 必须能够访问源文件或播放器（也就是说，源文件或播放器不能[被 robots.txt 屏蔽](https://support.google.com/webmasters/answer/6062608)、要求登录或因其他原因无法供 Googlebot 访问）。我们不支持需要通过流协议下载视频源的元文件。
* 所有文件都必须可供 Googlebot 访问。如果您想阻止垃圾内容发布者访问在 `<player_loc>` 或 `<content_loc>` 网址上的视频内容，请[验证访问您服务器的任何漫游器是否确实为 Googlebot](https://support.google.com/webmasters/answer/80553)。
* 确保 robots.txt 文件未屏蔽每个站点地图条目中包含的任何项（包括托管网页网址、视频网址和缩略图网址）。[详细了解 robots.txt。](https://support.google.com/webmasters/answer/156449)
* Google 会验证您为每个视频提供的信息是否与网站上的内容相符。如果不相符，可能不会将您的视频编入索引。
* 您可以在一个站点地图中指定来自不同网站的网页。所有网站（包括包含您的站点地图的网站）都必须通过 Search Console 进行验证。[详细了解如何管理多个网站的站点地图。](https://support.google.com/webmasters/answer/75712)

### 站点地图示例

以下是包含一个托管一个视频的网页的示例视频 Sitemap。此示例包含 Google 使用的所有标记。

```text
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:video="http://www.google.com/schemas/sitemap-video/1.1">
   <url>
     <loc>http://www.example.com/videos/some_video_landing_page.html</loc>
     <video:video>
       <video:thumbnail_loc>http://www.example.com/thumbs/123.jpg</video:thumbnail_loc>
       <video:title>适合夏季的烧烤排餐</video:title>
       <video:description>小安教您如何每次都能烤出美味牛排</video:description>
       <video:content_loc>
           http://streamserver.example.com/video123.mp4</video:content_loc>
       <video:player_loc>
         http://www.example.com/videoplayer.php?video=123</video:player_loc>
       <video:duration>600</video:duration>
       <video:expiration_date>2021-11-05T19:20:30+08:00</video:expiration_date>
       <video:rating>4.2</video:rating>
       <video:view_count>12345</video:view_count>
       <video:publication_date>2007-11-05T19:20:30+08:00</video:publication_date>
       <video:family_friendly>yes</video:family_friendly>
       <video:restriction relationship="allow">IE GB US CA</video:restriction>
       <video:price currency="EUR">1.99</video:price>
       <video:requires_subscription>yes</video:requires_subscription>
       <video:uploader
          info="http://www.example.com/users/grillymcgrillerson">GrillyMcGrillerson
       </video:uploader>
       <video:live>no</video:live>
     </video:video>
   </url>
</urlset>
```

### XML 命名空间

视频 Sitemap 标记在以下名称空间中定义：

```text
xmlns:video="http://www.google.com/schemas/sitemap-video/1.1"
```

### 视频 Sitemap 标记定义

您可以在 [rssboard.org](http://www.rssboard.org/media-rss) 上查找有关媒体站点地图的更多文档。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6807;&#x8BB0;</th>
      <th style="text-align:left">&#x662F;&#x5426;&#x5FC5;&#x9700;&#xFF1F;</th>
      <th style="text-align:left">&#x8BF4;&#x660E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>&lt;url&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x60A8;&#x7F51;&#x7AD9;&#x4E0A;&#x7684;&#x5355;&#x4E2A;&#x6258;&#x7BA1;&#x7F51;&#x9875;&#x7684;&#x7236;&#x7EA7;&#x6807;&#x8BB0;&#x3002;
        <a
        href="https://www.sitemaps.org/protocol.html">&#x7531;&#x57FA;&#x672C;&#x7AD9;&#x70B9;&#x5730;&#x56FE;&#x683C;&#x5F0F;&#x5B9A;&#x4E49;&#x3002;</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;loc&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x5728;&#x5176;&#x4E2D;&#x6258;&#x7BA1;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x89C6;&#x9891;&#x7684;&#x6258;&#x7BA1;&#x7F51;&#x9875;&#x3002;&#x7528;&#x6237;&#x70B9;&#x51FB;
          Google &#x641C;&#x7D22;&#x4E2D;&#x7684;&#x89C6;&#x9891;&#x7ED3;&#x679C;&#x540E;&#xFF0C;&#x7CFB;&#x7EDF;&#x4F1A;&#x5C06;&#x4ED6;&#x4EEC;&#x8F6C;&#x5230;&#x6B64;&#x7F51;&#x9875;&#x3002;&#x6B64;&#x7F51;&#x5740;&#x5728;&#x7AD9;&#x70B9;&#x5730;&#x56FE;&#x4E2D;&#x5FC5;&#x987B;&#x662F;&#x552F;&#x4E00;&#x7684;&#x3002;
          <a
          href="https://www.sitemaps.org/protocol.html">&#x7531;&#x57FA;&#x672C;&#x7AD9;&#x70B9;&#x5730;&#x56FE;&#x683C;&#x5F0F;&#x5B9A;&#x4E49;&#x3002;</a>
        </p>
        <p><b>&#x5BF9;&#x4E8E;&#x5355;&#x4E2A;&#x7F51;&#x9875;&#x4E0A;&#x7684;&#x591A;&#x4E2A;&#x89C6;&#x9891;</b>&#xFF0C;&#x4E3A;&#x8BE5;&#x7F51;&#x9875;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;
          &lt;<code>loc&gt;</code> &#x6807;&#x8BB0;&#xFF0C;&#x5E76;&#x4E3A;&#x8BE5;&#x7F51;&#x9875;&#x4E0A;&#x7684;&#x6BCF;&#x4E2A;&#x89C6;&#x9891;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x5B50;&#x7EA7; <code>&lt;video&gt;</code> &#x5143;&#x7D20;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:video&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left"><code>&lt;loc&gt;</code> &#x6240;&#x6307;&#x5B9A;&#x7F51;&#x9875;&#x4E0A;&#x7684;&#x5355;&#x4E2A;&#x89C6;&#x9891;&#x7684;&#x6240;&#x6709;&#x76F8;&#x5173;&#x4FE1;&#x606F;&#x7684;&#x7236;&#x7EA7;&#x5143;&#x7D20;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:thumbnail_loc&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x6307;&#x5411;&#x89C6;&#x9891;&#x7F29;&#x7565;&#x56FE;&#x6587;&#x4EF6;&#x7684;&#x7F51;&#x5740;&#x3002;
        <a
        href="https://support.google.com/webmasters/answer/156442#thumbnails">&#x8BF7;&#x53C2;&#x9605;&#x7F29;&#x7565;&#x56FE;&#x8981;&#x6C42;&#x3002;</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:title&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x6807;&#x9898;&#x3002;&#x6240;&#x6709; HTML &#x5B9E;&#x4F53;&#x90FD;&#x5E94;&#x8FDB;&#x884C;&#x8F6C;&#x4E49;&#x6216;&#x8005;&#x5C01;&#x88C5;&#x5728;&#x4E00;&#x4E2A;
        <a
        href="http://wikipedia.org/wiki/CDATA"><code>CDATA</code> &#x5757;</a>&#x4E2D;&#x3002;&#x5EFA;&#x8BAE;&#x6B64;&#x6807;&#x9898;&#x4E0E;&#x7F51;&#x9875;&#x4E0A;&#x663E;&#x793A;&#x7684;&#x89C6;&#x9891;&#x6807;&#x9898;&#x4E00;&#x81F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:description&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x7684;&#x8BF4;&#x660E;&#x3002;&#x4E0D;&#x5F97;&#x8D85;&#x8FC7;
        2048 &#x4E2A;&#x5B57;&#x7B26;&#x3002;&#x6240;&#x6709; HTML &#x5B9E;&#x4F53;&#x90FD;&#x5E94;&#x8FDB;&#x884C;&#x8F6C;&#x4E49;&#x6216;&#x8005;&#x5C01;&#x88C5;&#x5728;&#x4E00;&#x4E2A;
        <a
        href="http://wikipedia.org/wiki/CDATA"><code>CDATA</code> &#x5757;</a>&#x4E2D;&#x3002;&#x5FC5;&#x987B;&#x4E0E;&#x7F51;&#x9875;&#x4E0A;&#x663E;&#x793A;&#x7684;&#x8BF4;&#x660E;&#x4E00;&#x81F4;&#xFF08;&#x4E0D;&#x5FC5;&#x9010;&#x5B57;&#x5339;&#x914D;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:content_loc&gt;</code>
      </td>
      <td style="text-align:left"><em><b>&#x5FC5;&#x9700;&#xFF1A;</b></em><b><br />video:content_loc<br /></b><em><b>&#x6216;</b></em><b><br />video:player_loc</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5411;&#x5B9E;&#x9645;&#x89C6;&#x9891;&#x5A92;&#x4F53;&#x6587;&#x4EF6;&#x7684;&#x7F51;&#x5740;&#x3002;
          <a
          href="https://support.google.com/webmasters/answer/156442#file-types">&#x5E94;&#x91C7;&#x7528;&#x5176;&#x4E2D;&#x4E00;&#x79CD;&#x652F;&#x6301;&#x7684;&#x683C;&#x5F0F;&#x3002;</a>
        </p>
        <p>HTML &#x4E0D;&#x662F;&#x652F;&#x6301;&#x7684;&#x683C;&#x5F0F;&#x3002;&#x5141;&#x8BB8;&#x4F7F;&#x7528;
          Flash&#xFF0C;&#x4F46;&#x5927;&#x591A;&#x6570;&#x79FB;&#x52A8;&#x5E73;&#x53F0;&#x5DF2;&#x4E0D;&#x518D;&#x652F;&#x6301;
          Flash&#xFF0C;&#x56E0;&#x6B64;&#x53EF;&#x80FD;&#x4E0D;&#x592A;&#x7ECF;&#x5E38;&#x88AB;&#x7F16;&#x5165;&#x7D22;&#x5F15;&#x3002;</p>
        <p>&#x4E0D;&#x5F97;&#x4E0E; <code>&lt;loc&gt;</code> &#x7F51;&#x5740;&#x76F8;&#x540C;&#x3002;</p>
        <p>&#x76F8;&#x5F53;&#x4E8E;&#x7ED3;&#x6784;&#x5316;&#x6570;&#x636E;&#x4E2D;&#x7684;
          <a
          href="https://developers.google.com/search/docs/data-types/video"><code>VideoObject.contentUrl</code>
            </a>&#x3002;</p>
        <p><b>&#x6700;&#x4F73;&#x505A;&#x6CD5;</b>&#xFF1A;&#x5982;&#x679C;&#x60A8;&#x60F3;&#x9650;&#x5236;&#x5185;&#x5BB9;&#x8BBF;&#x95EE;&#x6743;&#x9650;&#xFF0C;&#x4F46;&#x4ECD;&#x7136;&#x5E0C;&#x671B;
          Googlebot &#x6293;&#x53D6;&#x8BE5;&#x5185;&#x5BB9;&#xFF0C;&#x8BF7;&#x786E;&#x4FDD;
          Googlebot &#x80FD;&#x591F;&#x4F7F;&#x7528;<a href="https://support.google.com/webmasters/answer/80553">&#x53CD;&#x5411; DNS &#x67E5;&#x627E;</a>&#x8BBF;&#x95EE;&#x60A8;&#x7684;&#x5185;&#x5BB9;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:player_loc&gt;</code>
      </td>
      <td style="text-align:left"><em><b>&#x5FC5;&#x9700;&#xFF1A;</b></em><b><br />video:content_loc<br /></b><em><b>&#x6216;</b></em><b><br />video:player_loc</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5411;<b>&#x7279;&#x5B9A;</b>&#x89C6;&#x9891;&#x7684;&#x64AD;&#x653E;&#x5668;&#x7684;&#x7F51;&#x5740;&#x3002;&#x901A;&#x5E38;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x662F;&#x6307; <code>&lt;embed&gt;</code> &#x6807;&#x8BB0;&#x7684; <code>src</code> &#x5143;&#x7D20;&#x4E2D;&#x7684;&#x4FE1;&#x606F;&#x3002;&#x4E0D;&#x5F97;&#x4E0E; <code>&lt;loc&gt;</code> &#x7F51;&#x5740;&#x76F8;&#x540C;&#x3002;&#x5BF9;&#x4E8E;
          YouTube &#x89C6;&#x9891;&#xFF0C;&#x4F7F;&#x7528;&#x6B64;&#x503C;&#x800C;&#x4E0D;&#x662F; <code>video:content_loc</code>&#x3002;&#x5B83;&#x76F8;&#x5F53;&#x4E8E;&#x7ED3;&#x6784;&#x5316;&#x6570;&#x636E;&#x4E2D;&#x7684;
          <a
          href="https://developers.google.com/search/docs/data-types/video"><code>VideoObject.embedUrl</code>
            </a>&#x3002;</p>
        <p>&#x4E0D;&#x5F97;&#x4E0E; <code>&lt;loc&gt;</code> &#x7F51;&#x5740;&#x76F8;&#x540C;&#x3002;</p>
        <p><b>&#x5C5E;&#x6027;&#xFF1A;</b>
        </p>
        <ul>
          <li><code>allow_embed</code> [&#x53EF;&#x9009;] Google &#x662F;&#x5426;&#x53EF;&#x4EE5;&#x5C06;&#x89C6;&#x9891;&#x5D4C;&#x5165;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#x4E2D;&#x3002;&#x5141;&#x8BB8;&#x7684;&#x503C;&#x4E3A; <code>yes</code> &#x6216; <code>no</code>&#x3002;</li>
        </ul>
        <p><b>&#x6700;&#x4F73;&#x505A;&#x6CD5;</b>&#xFF1A;&#x5982;&#x679C;&#x60A8;&#x60F3;&#x9650;&#x5236;&#x5185;&#x5BB9;&#x8BBF;&#x95EE;&#x6743;&#x9650;&#xFF0C;&#x4F46;&#x4ECD;&#x7136;&#x5E0C;&#x671B;
          Googlebot &#x6293;&#x53D6;&#x8BE5;&#x5185;&#x5BB9;&#xFF0C;&#x8BF7;&#x786E;&#x4FDD;
          Googlebot &#x80FD;&#x591F;&#x4F7F;&#x7528;<a href="https://support.google.com/webmasters/answer/80553">&#x53CD;&#x5411; DNS &#x67E5;&#x627E;</a>&#x8BBF;&#x95EE;&#x60A8;&#x7684;&#x5185;&#x5BB9;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:duration&gt;</code>
      </td>
      <td style="text-align:left">&#x63A8;&#x8350;</td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x7684;&#x65F6;&#x957F;&#xFF08;&#x4EE5;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;&#x503C;&#x5FC5;&#x987B;&#x4ECB;&#x4E8E; <code>1</code>&#xFF08;&#x542B;&#xFF09;&#x548C; <code>28800</code>&#xFF08;8
        &#x5C0F;&#x65F6;&#xFF0C;&#x542B;&#xFF09;&#x4E4B;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:expiration_date&gt;</code>
      </td>
      <td style="text-align:left">&#x9002;&#x7528;&#x65F6;&#x5EFA;&#x8BAE;&#x6DFB;&#x52A0;</td>
      <td style="text-align:left">
        <p>&#x89C6;&#x9891;&#x7684;&#x5931;&#x6548;&#x65E5;&#x671F;&#xFF08;&#x91C7;&#x7528;
          <a
          href="http://www.w3.org/TR/NOTE-datetime">W3C &#x683C;&#x5F0F;</a>&#xFF09;&#x3002;&#x5982;&#x679C;&#x60A8;&#x7684;&#x89C6;&#x9891;&#x4E0D;&#x4F1A;&#x5931;&#x6548;&#xFF0C;&#x5219;&#x4E0D;&#x8981;&#x6DFB;&#x52A0;&#x6B64;&#x6807;&#x8BB0;&#x3002;&#x5982;&#x679C;&#x5B58;&#x5728;&#x6B64;&#x6807;&#x8BB0;&#xFF0C;&#x5219;&#x5728;&#x6B64;&#x65E5;&#x671F;&#x4E4B;&#x540E;&#xFF0C;Google
            &#x641C;&#x7D22;&#x5C06;&#x4E0D;&#x4F1A;&#x663E;&#x793A;&#x60A8;&#x7684;&#x89C6;&#x9891;&#x3002;</p>
        <p>&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A;&#x5B8C;&#x6574;&#x65E5;&#x671F;
          (<code>YYYY-MM-DD</code>)&#xFF0C;&#x6216;&#x5B8C;&#x6574;&#x65E5;&#x671F;&#x52A0;&#x65F6;&#x3001;&#x5206;&#x548C;&#x79D2;&#x4EE5;&#x53CA;&#x65F6;&#x533A; <code>(YYYY-MM-DDThh:mm:ss+TZD</code>)&#x3002;</p>
        <p><b>&#x793A;&#x4F8B;</b>&#xFF1A;<code>2012-07-16T19:20:30+08:00</code>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:rating&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x7684;&#x8BC4;&#x5206;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A;&#x4ECB;&#x4E8E;
        0.0&#xFF08;&#x4E0B;&#x9650;&#xFF0C;&#x542B;&#xFF09;&#x5230; 5.0&#xFF08;&#x4E0A;&#x9650;&#xFF0C;&#x542B;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x6D6E;&#x70B9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:view_count&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x7684;&#x89C2;&#x770B;&#x6B21;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:publication_date&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x7B2C;&#x4E00;&#x6B21;&#x53D1;&#x5E03;&#x89C6;&#x9891;&#x7684;&#x65E5;&#x671F;&#xFF08;&#x91C7;&#x7528;
          <a
          href="http://www.w3.org/TR/NOTE-datetime">W3C &#x683C;&#x5F0F;</a>&#xFF09;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A;&#x5B8C;&#x6574;&#x65E5;&#x671F;
            (<code>YYYY-MM-DD</code>)&#xFF0C;&#x6216;&#x5B8C;&#x6574;&#x65E5;&#x671F;&#x52A0;&#x65F6;&#x3001;&#x5206;&#x548C;&#x79D2;&#x4EE5;&#x53CA;&#x65F6;&#x533A; <code>(YYYY-MM-DDThh:mm:ss+TZD</code>)&#x3002;</p>
        <p><b>&#x793A;&#x4F8B;</b>&#xFF1A;<code>2007-07-16T19:20:30+08:00</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:family_friendly&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p><code>yes</code>&#xFF08;&#x6216;&#x4E0D;&#x6DFB;&#x52A0;&#xFF09;&#xFF0C;&#x5982;&#x679C;&#x89C6;&#x9891;&#x53EF;&#x4EE5;&#x5728;&#x5F00;&#x542F;&#x5B89;&#x5168;&#x641C;&#x7D22;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x64AD;&#x653E;&#x3002;</p>
        <p><code>no</code>&#xFF0C;&#x5982;&#x679C;&#x89C6;&#x9891;&#x53EA;&#x80FD;&#x5728;&#x5173;&#x95ED;&#x5B89;&#x5168;&#x641C;&#x7D22;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x64AD;&#x653E;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:restriction&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x662F;&#x5426;&#x5728;&#x6765;&#x81EA;&#x7279;&#x5B9A;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x7684;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#x4E2D;&#x663E;&#x793A;&#x6216;&#x9690;&#x85CF;&#x60A8;&#x7684;&#x89C6;&#x9891;&#x3002;</p>
        <p>&#x4EE5; <a href="http://wikipedia.org/wiki/ISO_3166">ISO 3166 &#x683C;&#x5F0F;</a>&#x6307;&#x5B9A;&#x7528;&#x7A7A;&#x683C;&#x9694;&#x5F00;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x4EE3;&#x7801;&#x5217;&#x8868;&#x3002;&#x6BCF;&#x4E2A;&#x89C6;&#x9891;&#x53EA;&#x80FD;&#x4F7F;&#x7528;&#x4E00;&#x4E2A; <code>&lt;video:restriction&gt;</code> &#x6807;&#x8BB0;&#x3002;&#x5982;&#x679C;&#x6CA1;&#x6709; <code>&lt;video:restriction&gt;</code> &#x6807;&#x8BB0;&#xFF0C;Google
          &#x4F1A;&#x5047;&#x5B9A;&#x8BE5;&#x89C6;&#x9891;&#x53EF;&#x5728;&#x6240;&#x6709;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x64AD;&#x653E;&#x3002;<em>&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x6B64;&#x6807;&#x8BB0;&#x4EC5;&#x4F1A;&#x5F71;&#x54CD;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#xFF1B;&#x5B83;&#x4E0D;&#x4F1A;&#x963B;&#x6B62;&#x7528;&#x6237;&#x901A;&#x8FC7;&#x5176;&#x4ED6;&#x65B9;&#x5F0F;&#x5728;&#x53D7;&#x9650;&#x5236;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x67E5;&#x627E;&#x6216;&#x64AD;&#x653E;&#x60A8;&#x7684;&#x89C6;&#x9891;&#x3002;</em>
          <a
          href="https://support.google.com/webmasters/answer/156442#restrict_by_country">&#x8BE6;&#x7EC6;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5E94;&#x7528;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x9650;&#x5236;&#x3002;</a>
        </p>
        <p><b>&#x5C5E;&#x6027;&#xFF1A;</b>
        </p>
        <ul>
          <li><code>relationship</code> [&#x5FC5;&#x9700;] &#x89C6;&#x9891;&#x5728;&#x6307;&#x5B9A;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x7684;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#x4E2D;&#x662F;&#x5141;&#x8BB8;&#x663E;&#x793A;&#x8FD8;&#x662F;&#x62D2;&#x7EDD;&#x663E;&#x793A;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A; <code>allow</code> &#x6216; <code>deny</code>&#x3002;&#x5982;&#x679C;&#x662F; <code>allow</code>&#xFF0C;&#x5219;&#x5141;&#x8BB8;&#x5728;&#x5DF2;&#x5217;&#x51FA;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x663E;&#x793A;&#xFF0C;&#x62D2;&#x7EDD;&#x5728;&#x672A;&#x5217;&#x51FA;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x663E;&#x793A;&#xFF1B;&#x5982;&#x679C;&#x662F; <code>deny</code>&#xFF0C;&#x5219;&#x62D2;&#x7EDD;&#x5728;&#x5DF2;&#x5217;&#x51FA;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x663E;&#x793A;&#xFF0C;&#x5141;&#x8BB8;&#x5728;&#x672A;&#x5217;&#x51FA;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x663E;&#x793A;&#x3002;</li>
        </ul>
        <p><b>&#x793A;&#x4F8B;</b>&#xFF1A;&#x672C;&#x793A;&#x4F8B;&#x4EC5;&#x5141;&#x8BB8;&#x5728;&#x52A0;&#x62FF;&#x5927;&#x548C;&#x58A8;&#x897F;&#x54E5;&#x663E;&#x793A;&#x89C6;&#x9891;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#xFF1A;</p>
        <p><code>&lt;video:restriction relationship=&quot;allow&quot;&gt;CA MX&lt;/video:restriction&gt;</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:platform&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x662F;&#x5426;&#x5728;&#x6307;&#x5B9A;&#x7C7B;&#x578B;&#x7684;&#x5E73;&#x53F0;&#x4E0A;&#x7684;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#x4E2D;&#x663E;&#x793A;&#x6216;&#x9690;&#x85CF;&#x60A8;&#x7684;&#x89C6;&#x9891;&#x3002;&#x8FD9;&#x662F;&#x7528;&#x7A7A;&#x683C;&#x9694;&#x5F00;&#x7684;&#x5E73;&#x53F0;&#x7C7B;&#x578B;&#x5217;&#x8868;&#x3002;<em>&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x6B64;&#x6807;&#x8BB0;&#x4EC5;&#x4F1A;&#x5F71;&#x54CD;&#x6307;&#x5B9A;&#x8BBE;&#x5907;&#x7C7B;&#x578B;&#x4E0A;&#x7684;&#x641C;&#x7D22;&#x7ED3;&#x679C;&#xFF1B;&#x5B83;&#x4E0D;&#x4F1A;&#x963B;&#x6B62;&#x7528;&#x6237;&#x5728;&#x53D7;&#x9650;&#x5E73;&#x53F0;&#x4E0A;&#x64AD;&#x653E;&#x60A8;&#x7684;&#x89C6;&#x9891;&#x3002;</em>
        </p>
        <p>&#x5BF9;&#x4E8E;&#x6BCF;&#x4E2A;&#x89C6;&#x9891;&#xFF0C;&#x53EA;&#x80FD;&#x663E;&#x793A;&#x4E00;&#x4E2A; <code>&lt;video:platform&gt;</code> &#x6807;&#x8BB0;&#x3002;&#x5982;&#x679C;&#x6CA1;&#x6709; <code>&lt;video:platform&gt;</code> &#x6807;&#x8BB0;&#xFF0C;Google
          &#x4F1A;&#x5047;&#x5B9A;&#x8BE5;&#x89C6;&#x9891;&#x53EF;&#x4EE5;&#x5728;&#x6240;&#x6709;&#x5E73;&#x53F0;&#x4E0A;&#x64AD;&#x653E;&#x3002;
          <a
          href="https://support.google.com/webmasters/answer/156442#restrict_by_platform">&#x8BE6;&#x7EC6;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x5E94;&#x7528;&#x5E73;&#x53F0;&#x9650;&#x5236;&#x3002;</a>
        </p>
        <p><b>&#x652F;&#x6301;&#x7684;&#x503C;</b>&#xFF1A;</p>
        <ul>
          <li><code>web</code> - &#x684C;&#x9762;&#x8BBE;&#x5907;&#x548C;&#x7B14;&#x8BB0;&#x672C;&#x7535;&#x8111;&#x4E2D;&#x7684;&#x4F20;&#x7EDF;&#x8BA1;&#x7B97;&#x673A;&#x6D4F;&#x89C8;&#x5668;&#x3002;</li>
          <li><code>mobile</code> - &#x79FB;&#x52A8;&#x6D4F;&#x89C8;&#x5668;&#xFF0C;&#x4F8B;&#x5982;&#x624B;&#x673A;&#x6216;&#x5E73;&#x677F;&#x7535;&#x8111;&#x4E2D;&#x7684;&#x6D4F;&#x89C8;&#x5668;&#x3002;</li>
          <li><code>tv</code> - &#x7535;&#x89C6;&#x6D4F;&#x89C8;&#x5668;&#xFF0C;&#x4F8B;&#x5982;
            Android TV &#x8BBE;&#x5907;&#x548C;&#x6E38;&#x620F;&#x673A;&#x4E0A;&#x7684;&#x6D4F;&#x89C8;&#x5668;&#x3002;</li>
        </ul>
        <p><b>&#x5C5E;&#x6027;&#xFF1A;</b>
        </p>
        <ul>
          <li><code>relationship</code> [&#x5FC5;&#x9700;] &#x7528;&#x4E8E;&#x6307;&#x5B9A;&#x89C6;&#x9891;&#x5728;&#x6307;&#x5B9A;&#x5E73;&#x53F0;&#x4E0A;&#x662F;&#x62D2;&#x7EDD;&#x663E;&#x793A;&#x8FD8;&#x662F;&#x5141;&#x8BB8;&#x663E;&#x793A;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A; <code>allow</code> &#x6216; <code>deny</code>&#x3002;&#x5982;&#x679C;&#x662F; <code>allow</code>&#xFF0C;&#x5219;&#x62D2;&#x7EDD;&#x5728;&#x6240;&#x6709;&#x672A;&#x5217;&#x51FA;&#x7684;&#x5E73;&#x53F0;&#x4E0A;&#x663E;&#x793A;&#xFF1B;&#x5982;&#x679C;&#x662F; <code>deny</code>&#xFF0C;&#x5219;&#x5141;&#x8BB8;&#x5728;&#x6240;&#x6709;&#x672A;&#x5217;&#x51FA;&#x7684;&#x5E73;&#x53F0;&#x4E0A;&#x663E;&#x793A;&#x3002;</li>
        </ul>
        <p><b>&#x793A;&#x4F8B;</b>&#xFF1A;&#x4EE5;&#x4E0B;&#x793A;&#x4F8B;&#x5141;&#x8BB8;&#x5411;&#x4F7F;&#x7528;&#x4F20;&#x7EDF;&#x8BA1;&#x7B97;&#x673A;&#x6D4F;&#x89C8;&#x5668;&#x6216;&#x7535;&#x89C6;&#x6D4F;&#x89C8;&#x5668;&#x7684;&#x7528;&#x6237;&#x663E;&#x793A;&#x89C6;&#x9891;&#xFF0C;&#x4F46;&#x4E0D;&#x5141;&#x8BB8;&#x5411;&#x4F7F;&#x7528;&#x79FB;&#x52A8;&#x6D4F;&#x89C8;&#x5668;&#x7684;&#x7528;&#x6237;&#x663E;&#x793A;&#x89C6;&#x9891;&#xFF1A;
          <br
          /><code>&lt;video:platform relationship=&quot;allow&quot;&gt;web tv&lt;/video:platform&gt;</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:price&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x4E0B;&#x8F7D;&#x6216;&#x89C2;&#x770B;&#x89C6;&#x9891;&#x9700;&#x8981;&#x652F;&#x4ED8;&#x7684;&#x8D39;&#x7528;&#x3002;&#x5982;&#x679C;&#x662F;&#x514D;&#x8D39;&#x89C6;&#x9891;&#xFF0C;&#x5219;&#x4E0D;&#x8981;&#x6DFB;&#x52A0;&#x6B64;&#x6807;&#x8BB0;&#x3002;&#x60A8;&#x53EF;&#x4EE5;&#x5217;&#x51FA;&#x591A;&#x4E2A; <code>&lt;video:price&gt;</code> &#x5143;&#x7D20;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x7528;&#x4E8E;&#x6307;&#x5B9A;&#x591A;&#x4E2A;&#x5E01;&#x79CD;&#x3001;&#x8D2D;&#x4E70;&#x9009;&#x9879;&#x6216;&#x5206;&#x8FA8;&#x7387;&#xFF09;&#x3002;</p>
        <p><b>&#x5C5E;&#x6027;&#xFF1A;</b>
        </p>
        <ul>
          <li><code>currency</code> [&#x5FC5;&#x9700;] &#x4EE5; <a href="http://en.wikipedia.org/wiki/ISO_4217">ISO 4217 &#x683C;&#x5F0F;</a>&#x6307;&#x5B9A;&#x5E01;&#x79CD;&#x3002;</li>
          <li><code>type</code> [&#x53EF;&#x9009;] &#x6307;&#x5B9A;&#x8D2D;&#x4E70;&#x9009;&#x9879;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A; <code>rent</code> &#x548C; <code>own</code>&#x3002;&#x5982;&#x679C;&#x672A;&#x6307;&#x5B9A;&#xFF0C;&#x7CFB;&#x7EDF;&#x4F1A;&#x4F7F;&#x7528;&#x9ED8;&#x8BA4;&#x503C; <code>own</code>&#x3002;</li>
          <li><code>resolution</code> [&#x53EF;&#x9009;] &#x6307;&#x5B9A;&#x8D2D;&#x4E70;&#x7248;&#x672C;&#x7684;&#x5206;&#x8FA8;&#x7387;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A; <code>hd</code> &#x548C; <code>sd</code>&#x3002;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:requires_subscription&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x6307;&#x660E;&#x662F;&#x5426;&#x9700;&#x8981;&#x8BA2;&#x9605;&#xFF08;&#x6536;&#x8D39;&#x6216;&#x514D;&#x8D39;&#xFF09;&#x624D;&#x80FD;&#x89C2;&#x770B;&#x89C6;&#x9891;&#x3002;&#x5141;&#x8BB8;&#x7684;&#x503C;&#x4E3A; <code>yes</code> &#x6216; <code>no</code>&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:uploader&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x89C6;&#x9891;&#x4E0A;&#x4F20;&#x8005;&#x7684;&#x540D;&#x79F0;&#x3002;&#x6BCF;&#x4E2A;&#x89C6;&#x9891;&#x53EA;&#x80FD;&#x6709;&#x4E00;&#x4E2A; <code>&lt;video:uploader&gt;</code>&#x3002;&#x5B57;&#x7B26;&#x4E32;&#x503C;&#xFF0C;&#x6700;&#x591A;
          255 &#x4E2A;&#x5B57;&#x7B26;&#x3002;</p>
        <p><b>&#x5C5E;&#x6027;&#xFF1A;</b>
        </p>
        <ul>
          <li><code>info</code> [&#x53EF;&#x9009;] &#x6307;&#x5B9A;&#x5176;&#x4E2D;&#x5305;&#x542B;&#x6709;&#x5173;&#x6B64;&#x4E0A;&#x4F20;&#x8005;&#x7684;&#x5176;&#x4ED6;&#x4FE1;&#x606F;&#x7684;&#x7F51;&#x9875;&#x5BF9;&#x5E94;&#x7684;&#x7F51;&#x5740;&#x3002;&#x8BE5;&#x7F51;&#x5740;&#x5FC5;&#x987B;&#x4E0E; <code>&lt;loc&gt;</code> &#x6807;&#x8BB0;&#x4F4D;&#x4E8E;&#x540C;&#x4E00;&#x4E2A;&#x7F51;&#x57DF;&#x4E2D;&#x3002;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:live&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x6307;&#x660E;&#x89C6;&#x9891;&#x662F;&#x5426;&#x4E3A;&#x76F4;&#x64AD;&#x89C6;&#x9891;&#x3002;&#x652F;&#x6301;&#x7684;&#x503C;&#x4E3A; <code>yes</code> &#x6216; <code>no</code>&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:tag&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x7528;&#x4E8E;&#x63CF;&#x8FF0;&#x89C6;&#x9891;&#x7684;&#x4EFB;&#x610F;&#x5B57;&#x7B26;&#x4E32;&#x6807;&#x8BB0;&#x3002;&#x6807;&#x8BB0;&#x901A;&#x5E38;&#x662F;&#x4E0E;&#x89C6;&#x9891;&#x6216;&#x5185;&#x5BB9;&#x7247;&#x6BB5;&#x76F8;&#x5173;&#x8054;&#x7684;&#x5173;&#x952E;&#x6982;&#x5FF5;&#x7684;&#x6781;&#x7B80;&#x77ED;&#x8BF4;&#x660E;&#x3002;&#x4E00;&#x4E2A;&#x89C6;&#x9891;&#x53EF;&#x4EE5;&#x6709;&#x591A;&#x4E2A;&#x6807;&#x8BB0;&#xFF0C;&#x5C3D;&#x7BA1;&#x5B83;&#x53EF;&#x80FD;&#x53EA;&#x5C5E;&#x4E8E;&#x4E00;&#x4E2A;&#x7C7B;&#x522B;&#x3002;&#x4F8B;&#x5982;&#xFF0C;&#x6709;&#x5173;&#x70E7;&#x70E4;&#x98DF;&#x7269;&#x7684;&#x89C6;&#x9891;&#x53EF;&#x80FD;&#x5C5E;&#x4E8E;&#x201C;&#x70E7;&#x70E4;&#x201D;&#x7C7B;&#x522B;&#xFF0C;&#x4F46;&#x53EF;&#x4EE5;&#x5E26;&#x6709;&#x201C;&#x725B;&#x6392;&#x201D;&#x3001;&#x201C;&#x8089;&#x7C7B;&#x201D;&#x3001;&#x201C;&#x590F;&#x5B63;&#x201D;&#x548C;&#x201C;&#x5BA4;&#x5916;&#x201D;&#x6807;&#x8BB0;&#x3002;&#x8BF7;&#x4E3A;&#x6BCF;&#x4E2A;&#x4E0E;&#x89C6;&#x9891;&#x76F8;&#x5173;&#x7684;&#x6807;&#x8BB0;&#x521B;&#x5EFA;&#x65B0;&#x7684; <code>&lt;video:tag&gt;</code> &#x5143;&#x7D20;&#x3002;&#x6700;&#x591A;&#x5141;&#x8BB8;&#x4F7F;&#x7528;
        32 &#x4E2A;&#x6807;&#x8BB0;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:category&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x6240;&#x5C5E;&#x5BBD;&#x6CDB;&#x7C7B;&#x522B;&#x7684;&#x7B80;&#x77ED;&#x8BF4;&#x660E;&#x3002;&#x8FD9;&#x662F;&#x4E00;&#x4E2A;&#x4E0D;&#x8D85;&#x8FC7;
        256 &#x4E2A;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;&#x4E00;&#x822C;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x7C7B;&#x522B;&#x662F;&#x6307;&#x6309;&#x4E3B;&#x9898;&#x5BF9;&#x5185;&#x5BB9;&#x8FDB;&#x884C;&#x7684;&#x5927;&#x81F4;&#x5206;&#x7EC4;&#x3002;&#x901A;&#x5E38;&#xFF0C;&#x4E00;&#x4E2A;&#x89C6;&#x9891;&#x53EA;&#x5C5E;&#x4E8E;&#x4E00;&#x4E2A;&#x7C7B;&#x522B;&#x3002;&#x4F8B;&#x5982;&#xFF0C;&#x6709;&#x5173;&#x70F9;&#x996A;&#x7684;&#x7F51;&#x7AD9;&#x53EF;&#x4EE5;&#x6709;&#x201C;&#x94C1;&#x677F;&#x70E7;&#x201D;&#x3001;&#x201C;&#x70D8;&#x70E4;&#x201D;&#x548C;&#x201C;&#x70E7;&#x70E4;&#x201D;&#x7B49;&#x7C7B;&#x522B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;video:gallery_loc&gt;</code>
      </td>
      <td style="text-align:left"><em>&#x672A;&#x4F7F;&#x7528;</em>
      </td>
      <td style="text-align:left">&#x5F53;&#x524D;&#x672A;&#x4F7F;&#x7528;&#x3002;</td>
    </tr>
  </tbody>
</table>

### 站点地图替代方案

虽然 Google 建议您使用视频 Sitemap 和 [schema.org 的 VideoObject](https://support.google.com/webmasters/answer/156442#videoobject) 来标记您的视频，但我们也支持 mRSS Feed。

### mRSS

Google 支持 [mRSS](http://www.rssboard.org/media-rss)，它是一个用于补充 [RSS 2.0](http://cyber.law.harvard.edu/rss/rss.html) 的元素功能的 RSS 模块。mRSS Feed 与视频 Sitemap 非常类似，可以像站点地图一样进行测试、提交和更新。

对于每个 mRSS Feed，未压缩时的文件大小都不得超过 50MB，且包含的视频条目不超过 5 万个。如果未压缩文件大于 50MB，或者视频数量超过 5 万个，您可以提交多个 mRSS Feed 和[站点地图索引文件](https://support.google.com/webmasters/answer/71453)。站点地图索引可以包含 mRSS Feed。**RSS 与 mRSS** - mRSS 是用于整合多媒体文件的 RSS 扩展。它对内容的描述比 RSS 标准要详细得多。

#### mRSS 示例

下面是 mRSS 条目的示例，其中提供了 Google 使用的所有主要标记。该示例包含了 `<dcterms:type>live-video</dcterms:type>`，您可以使用该标记标识直播视频。

```text
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:media="http://search.yahoo.com/mrss/" xmlns:dcterms="http://purl.org/dc/terms/">
<channel>
<title>示例 MRSS</title>
<link>http://www.example.com/examples/mrss/</link>
<description>MRSS 示例</description>
  <item xmlns:media="http://search.yahoo.com/mrss/" xmlns:dcterms="http://purl.org/dc/terms/">
    <link>http://www.example.com/examples/mrss/example.html</link>
    <media:content url="http://www.example.com/examples/mrss/example.flv" fileSize="405321"
      type="video/x-flv" height="240" width="320" duration="120" medium="video" isDefault="true">
      <media:player url="http://www.example.com/shows/example/video.swf?flash_params" />
      <media:title>适合夏季的烧烤排餐</media:title>
      <media:description>每次都能烤出美味牛排</media:description>
      <media:thumbnail url="http://www.example.com/examples/mrss/example.png" height="120" width="160"/>
      <media:price price="19.99" currency="EUR" />
      <media:price type="subscription" />
    </media:content>
    <media:restriction relationship="allow" type="country">us ca</media:restriction>
    <dcterms:valid xmlns:dcterms="http://purl.org/dc/terms/">end=2020-10-15T00:00+01:00; scheme=W3C-DTF</dcterms:valid>
    <dcterms:type>live-video</dcterms:type>
  </item>
</channel>
</rss>
```

#### mRSS 标记

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6807;&#x8BB0;</th>
      <th style="text-align:left">&#x662F;&#x5426;&#x5FC5;&#x9700;&#xFF1F;</th>
      <th style="text-align:left">&#x8BF4;&#x660E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>&lt;media:content&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">
        <p>&#x9644;&#x4E0A;&#x89C6;&#x9891;&#x7684;&#x76F8;&#x5173;&#x4FE1;&#x606F;&#x3002;</p>
        <p>&#x5C5E;&#x6027;&#xFF1A;</p>
        <ul>
          <li><code>medium</code> [&#x5FC5;&#x9700;] &#x5185;&#x5BB9;&#x7684;&#x7C7B;&#x578B;&#x3002;&#x6B64;&#x5C5E;&#x6027;&#x5E94;&#x8BBE;&#x7F6E;&#x4E3A; <code>video</code>&#x3002;</li>
          <li><code>url</code> [&#x5FC5;&#x9700;] &#x6307;&#x5411;&#x539F;&#x59CB;&#x89C6;&#x9891;&#x5185;&#x5BB9;&#x7684;&#x76F4;&#x63A5;&#x7F51;&#x5740;&#x3002;<b>&#x5982;&#x679C;&#x672A;&#x6307;&#x5B9A;&#x6B64;&#x5C5E;&#x6027;&#xFF0C;&#x5219;&#x5FC5;&#x987B;&#x6307;&#x5B9A; <code>&lt;media:player&gt;</code> &#x6807;&#x8BB0;&#x3002;</b>
          </li>
          <li><code>duration</code> [&#x53EF;&#x9009;&#xFF0C;&#x4F46;&#x5EFA;&#x8BAE;&#x4F7F;&#x7528;]
            &#x89C6;&#x9891;&#x7684;&#x65F6;&#x957F;&#xFF08;&#x4EE5;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</li>
        </ul>
        <p>&#x8981;&#x4E86;&#x89E3; <code>&lt;media:content&gt;</code> &#x6807;&#x8BB0;&#x7684;&#x6240;&#x6709;&#x5176;&#x4ED6;&#x53EF;&#x9009;&#x5C5E;&#x6027;&#x548C;&#x5B50;&#x5B57;&#x6BB5;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="http://www.rssboard.org/media-rss">mRSS &#x89C4;&#x8303;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;media:player&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x89C6;&#x60C5;&#x51B5;&#x800C;&#x5B9A;</b>
      </td>
      <td style="text-align:left">
        <p><b>&#x60A8;&#x4E0D;&#x80FD;&#x65E2;&#x4E0D;&#x6307;&#x5B9A; <code>&lt;media:player&gt;</code>&#xFF0C;&#x4E5F;&#x4E0D;&#x6307;&#x5B9A; <code>&lt;media:content&gt;</code> &#x4E2D;&#x7684; <code>url</code> &#x5C5E;&#x6027;&#x3002;</b>
        </p>
        <p>&#x6307;&#x5411;<b>&#x7279;&#x5B9A;</b>&#x89C6;&#x9891;&#x7684;&#x64AD;&#x653E;&#x5668;&#x7684;&#x7F51;&#x5740;&#x3002;&#x901A;&#x5E38;&#xFF0C;&#x8BE5;&#x7F51;&#x5740;&#x662F; <code>&lt;embed&gt;</code> &#x6807;&#x8BB0;&#x7684; <code>src</code> &#x5143;&#x7D20;&#x4E2D;&#x7684;&#x4FE1;&#x606F;&#xFF0C;&#x4E0D;&#x5E94;&#x8BE5;&#x4E0E; <code>&lt;loc&gt;</code> &#x6807;&#x8BB0;&#x7684;&#x5185;&#x5BB9;&#x76F8;&#x540C;&#x3002;&#x4E0D;&#x80FD;&#x4E0E; <code>&lt;link&gt;</code> &#x6807;&#x8BB0;&#x7684;&#x7F51;&#x5740;&#x76F8;&#x540C;&#x3002;<code>&lt;link&gt;</code> &#x5E94;&#x6307;&#x5411;&#x6258;&#x7BA1;&#x8BE5;&#x89C6;&#x9891;&#x7684;&#x7F51;&#x9875;&#x7684;&#x7F51;&#x5740;&#xFF0C;&#x800C;&#x6B64;&#x6807;&#x8BB0;&#x5E94;&#x6307;&#x5411;&#x64AD;&#x653E;&#x5668;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;media:title&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x6807;&#x9898;&#x3002;&#x4E0D;&#x5F97;&#x8D85;&#x8FC7;
        100 &#x4E2A;&#x5B57;&#x7B26;&#x3002;&#x6240;&#x6709; HTML &#x5B9E;&#x4F53;&#x90FD;&#x5E94;&#x8FDB;&#x884C;&#x8F6C;&#x4E49;&#x6216;&#x8005;&#x5C01;&#x88C5;&#x5728;&#x4E00;&#x4E2A;
        <a
        href="http://wikipedia.org/wiki/CDATA">CDATA &#x5757;</a>&#x4E2D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;media:description&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x89C6;&#x9891;&#x8BF4;&#x660E;&#x3002;&#x4E0D;&#x5F97;&#x8D85;&#x8FC7;
        2048 &#x4E2A;&#x5B57;&#x7B26;&#x3002;&#x6240;&#x6709; HTML &#x5B9E;&#x4F53;&#x90FD;&#x5E94;&#x8FDB;&#x884C;&#x8F6C;&#x4E49;&#x6216;&#x8005;&#x5C01;&#x88C5;&#x5728;&#x4E00;&#x4E2A;
        <a
        href="http://wikipedia.org/wiki/CDATA">CDATA &#x5757;</a>&#x4E2D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;media:thumbnail&gt;</code>
      </td>
      <td style="text-align:left"><b>&#x5FC5;&#x9700;</b>
      </td>
      <td style="text-align:left">&#x6307;&#x5411;&#x9884;&#x89C8;&#x7F29;&#x7565;&#x56FE;&#x7684;&#x7F51;&#x5740;&#x3002;
        <a
        href="https://support.google.com/webmasters/answer/156442#thumbnails">&#x8BF7;&#x53C2;&#x9605;&#x7F29;&#x7565;&#x56FE;&#x8981;&#x6C42;&#x3002;</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;dcterms:valid&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x89C6;&#x9891;&#x7684;&#x53D1;&#x5E03;&#x65E5;&#x671F;&#x548C;&#x5931;&#x6548;&#x65E5;&#x671F;&#x3002;
          <a
          href="https://www.dublincore.org/specifications/dublin-core/dcmi-terms/terms/valid/"><code>dcterms:valid</code> &#x7684;&#x5B8C;&#x6574;&#x89C4;&#x8303;</a>&#x3002;</p>
        <p><b>&#x793A;&#x4F8B;&#xFF1A;</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;media:restriction&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x53EF;&#x4EE5;&#x64AD;&#x653E;&#x89C6;&#x9891;&#x6216;&#x65E0;&#x6CD5;&#x64AD;&#x653E;&#x89C6;&#x9891;&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x5217;&#x8868;&#xFF0C;&#x5176;&#x4E2D;&#x5404;&#x9879;&#x7528;&#x7A7A;&#x683C;&#x9694;&#x5F00;&#x3002;&#x5141;&#x8BB8;&#x7684;&#x503C;&#x4E3A;&#x91C7;&#x7528;
          <a
          href="http://wikipedia.org/wiki/ISO_3166">ISO 3166 &#x683C;&#x5F0F;</a>&#x7684;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x4EE3;&#x7801;&#x3002;&#x5982;&#x679C;&#x6CA1;&#x6709; <code>&lt;media:restriction&gt;</code> &#x6807;&#x8BB0;&#xFF0C;Google
            &#x4F1A;&#x5047;&#x5B9A;&#x8BE5;&#x89C6;&#x9891;&#x53EF;&#x5728;&#x6240;&#x6709;&#x5730;&#x533A;&#x64AD;&#x653E;&#x3002;</p>
        <p><code>type</code> &#x662F;<b>&#x5FC5;&#x9700;&#x5C5E;&#x6027;</b>&#xFF0C;&#x5E94;&#x8BBE;&#x7F6E;&#x4E3A; <code>country</code>&#x3002;&#x4EC5;&#x652F;&#x6301;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x9650;&#x5236;&#x3002;</p>
        <p><code>relationship</code> &#x662F;<b>&#x5FC5;&#x9700;&#x5C5E;&#x6027;</b>&#xFF0C;&#x7528;&#x4E8E;&#x6307;&#x5B9A;&#x89C6;&#x9891;&#x5728;&#x6307;&#x5B9A;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x662F;&#x5426;&#x5141;&#x8BB8;&#x64AD;&#x653E;&#x3002;&#x5141;&#x8BB8;&#x7684;&#x503C;&#x4E3A; <code>allow</code> &#x6216; <code>deny</code>&#x3002;</p>
        <p><a href="https://support.google.com/webmasters/answer/156442#restrict_by_country">&#x8BE6;&#x7EC6;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x4F7F;&#x7528;&#x56FD;&#x5BB6;/&#x5730;&#x533A;&#x9650;&#x5236;&#x3002;</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;media:price&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">
        <p>&#x4E0B;&#x8F7D;&#x6216;&#x89C2;&#x770B;&#x89C6;&#x9891;&#x9700;&#x8981;&#x652F;&#x4ED8;&#x7684;&#x8D39;&#x7528;&#x3002;&#x5982;&#x679C;&#x662F;&#x514D;&#x8D39;&#x89C6;&#x9891;&#xFF0C;&#x8BF7;&#x52FF;&#x4F7F;&#x7528;&#x6B64;&#x6807;&#x8BB0;&#x3002;&#x60A8;&#x53EF;&#x4EE5;&#x5217;&#x51FA;&#x591A;&#x4E2A; <code>&lt;media:price&gt;</code> &#x5143;&#x7D20;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x7528;&#x4E8E;&#x6307;&#x5B9A;&#x591A;&#x4E2A;&#x5E01;&#x79CD;&#x6216;&#x8D2D;&#x4E70;&#x9009;&#x9879;&#xFF09;&#x3002;</p>
        <p><b>&#x5C5E;&#x6027;&#xFF1A;</b>
        </p>
        <ul>
          <li><code>currency</code> [&#x5FC5;&#x9700;] &#x4EE5; <a href="http://en.wikipedia.org/wiki/ISO_4217">ISO 4217 &#x683C;&#x5F0F;</a>&#x6307;&#x5B9A;&#x7684;&#x5E01;&#x79CD;&#x3002;</li>
          <li><code>type</code> [&#x5FC5;&#x9700;] &#x8D2D;&#x4E70;&#x9009;&#x9879;&#x3002;&#x5141;&#x8BB8;&#x7684;&#x503C;&#x4E3A; <code>rent</code>&#x3001;<code>purchase</code>&#x3001;<code>package</code> &#x548C; <code>subscription</code>&#x3002;</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

[完整的 mRSS 规范](http://www.rssboard.org/media-rss)包含更多可选标记、最佳做法和示例。创建 mRSS Feed 后，您便可以[测试和提交](https://support.google.com/webmasters/videosearch/testing.html)此 Feed，就像测试和提交视频 Sitemap 一样。

