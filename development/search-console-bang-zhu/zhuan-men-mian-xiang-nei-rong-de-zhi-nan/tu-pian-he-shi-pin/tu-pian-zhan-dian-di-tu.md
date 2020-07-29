# 图片站点地图

您可以向现有站点地图添加图片，或为您的图片创建单独的站点地图。向站点地图添加图片有助于 Google 发现我们可能无法通过其他方式找到的图片（例如，您的网站通过 JavaScript 代码链接的图片）。

### 最佳做法

* 遵循[发布图片的最佳做法](https://support.google.com/webmasters/answer/114016)和[网站站长指南](https://support.google.com/webmasters/answer/35769)。
* 您可以[添加图片元数据](https://support.google.com/webmasters/answer/9150868)，例如联系信息和许可信息，这些信息将显示在图片搜索结果中。

### 站点地图示例

以下所示是网页 `http://example.com/sample.html` 的站点地图条目，该网页包含两张图片。

```text
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
  <url>
    <loc>http://example.com/sample.html</loc>
    <image:image>
      <image:loc>http://example.com/image.jpg</image:loc>
    </image:image>
    <image:image>
      <image:loc>http://example.com/photo.jpg</image:loc>
    </image:image>
  </url> 
</urlset> 
```

使用上述示例中介绍的语法，您最多可以为每个网页列出 1000 张图片！

### 图片站点地图引用

#### XML 命名空间

图片标记在以下命名空间中定义：

```text
image:xmlns="http://www.google.com/schemas/sitemap-image/1.1"
```

#### 图片标记定义

以下站点地图标记专门适用于图片。您必须使用必需的标记；我们建议您使用可选标记提供更多信息和更好的用户体验。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6807;&#x8BB0;</th>
      <th style="text-align:left">&#x662F;&#x5426;&#x5FC5;&#x9700;</th>
      <th style="text-align:left">&#x8BF4;&#x660E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>&lt;image:image&gt;</code>
      </td>
      <td style="text-align:left">&#x662F;</td>
      <td style="text-align:left">&#x5305;&#x542B;&#x5355;&#x5F20;&#x56FE;&#x7247;&#x7684;&#x6240;&#x6709;&#x76F8;&#x5173;&#x4FE1;&#x606F;&#x3002;&#x6BCF;&#x4E2A; <code>&lt;url&gt;</code> &#x6807;&#x8BB0;&#x6700;&#x591A;&#x53EF;&#x5305;&#x542B;
        1000 <code>&lt;image:image&gt;</code> &#x6807;&#x8BB0;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;image:loc&gt;</code>
      </td>
      <td style="text-align:left">&#x662F;</td>
      <td style="text-align:left">
        <p>&#x56FE;&#x7247;&#x7684;&#x7F51;&#x5740;&#x3002;</p>
        <p>&#x5728;&#x67D0;&#x4E9B;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x56FE;&#x7247;&#x7F51;&#x5740;&#x53EF;&#x80FD;&#x4E0E;&#x60A8;&#x7684;&#x4E3B;&#x7F51;&#x7AD9;&#x4E0D;&#x5728;&#x540C;&#x4E00;&#x4E2A;&#x7F51;&#x57DF;&#x4E2D;&#x3002;&#x5373;&#x4FBF;&#x5982;&#x6B64;&#x4E5F;&#x4E0D;&#x5FC5;&#x62C5;&#x5FC3;&#xFF0C;&#x53EA;&#x8981;&#x8FD9;&#x4E24;&#x4E2A;&#x7F51;&#x57DF;&#x5728;
          Search Console &#x4E2D;&#x90FD;&#x5DF2;&#x9A8C;&#x8BC1;&#x5373;&#x53EF;&#x3002;&#x4F8B;&#x5982;&#xFF0C;&#x5F53;&#x60A8;&#x4F7F;&#x7528;&#x5185;&#x5BB9;&#x4F20;&#x9001;&#x7F51;&#x7EDC;&#xFF08;&#x5982;
          Google &#x534F;&#x4F5C;&#x5E73;&#x53F0;&#xFF09;&#x6258;&#x7BA1;&#x56FE;&#x7247;&#x65F6;&#xFF0C;&#x8BF7;&#x786E;&#x4FDD;&#x5728;
          Search Console &#x4E2D;&#x9A8C;&#x8BC1;&#x76F8;&#x5E94;&#x6258;&#x7BA1;&#x7F51;&#x7AD9;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x60A8;&#x5E94;&#x786E;&#x4FDD;
          <a
          href="https://support.google.com/webmasters/answer/75712">robots.txt</a>&#x6587;&#x4EF6;&#x5141;&#x8BB8;&#x6293;&#x53D6;&#x60F3;&#x8981;&#x7F16;&#x5165;&#x7D22;&#x5F15;&#x7684;&#x6240;&#x6709;&#x5185;&#x5BB9;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;image:caption&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x56FE;&#x7247;&#x7684;&#x8BF4;&#x660E;&#x6587;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;image:geo_location&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x56FE;&#x7247;&#x7684;&#x5730;&#x7406;&#x4F4D;&#x7F6E;&#x3002;&#x4F8B;&#x5982; <code>&lt;image:geo_location&gt;Limerick, Ireland&lt;/image:geo_location&gt;</code>&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;image:title&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x56FE;&#x7247;&#x7684;&#x6807;&#x9898;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>&lt;image:license&gt;</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;</td>
      <td style="text-align:left">&#x6307;&#x5411;&#x56FE;&#x7247;&#x8BB8;&#x53EF;&#x7684;&#x7F51;&#x5740;&#x3002;&#x5982;&#x679C;&#x9700;&#x8981;&#xFF0C;&#x60A8;&#x53EF;&#x4EE5;
        <a
        href="https://support.google.com/webmasters/answer/9150868">&#x6539;&#x7528;&#x56FE;&#x7247;&#x5143;&#x6570;&#x636E;</a>&#x3002;</td>
    </tr>
  </tbody>
</table>

