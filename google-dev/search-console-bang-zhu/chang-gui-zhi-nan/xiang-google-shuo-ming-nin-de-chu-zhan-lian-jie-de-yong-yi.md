# 向 Google 说明您的出站链接的用意

对于您网站上的某些出站链接，您可能想向 Google 说明您的网站与相应链接页之间的关系。为此，您应在 `<a>` 标记中使用下列 `rel` 属性值之一。

对于您认为无需说明用意、允许 Google 直接跟踪的常规链接，无需添加 `rel` 属性。**示例**：“我最喜欢的马是`<a href="https://horses.example.com/Palomino">`帕洛米诺马`</a>`。”

对于其他链接，请使用以下一个或多个值：

<table>
  <thead>
    <tr>
      <th style="text-align:left"><code>rel</code> &#x503C;</th>
      <th style="text-align:left">&#x8BF4;&#x660E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>rel=&quot;sponsored&quot;</code>
      </td>
      <td style="text-align:left">
        <p>&#x8BF7;&#x5C06;&#x5E7F;&#x544A;&#x94FE;&#x63A5;&#x6216;&#x4ED8;&#x8D39;&#x5C55;&#x793A;&#x4F4D;&#x7F6E;&#x94FE;&#x63A5;&#xFF08;&#x901A;&#x5E38;&#x79F0;&#x4E3A;&#x201C;&#x4ED8;&#x8D39;&#x94FE;&#x63A5;&#x201D;&#xFF09;&#x6807;&#x8BB0;&#x4E3A; <code>sponsored</code>&#x3002;
          <a
          href="https://support.google.com/webmasters/answer/66736">&#x8BE6;&#x7EC6;&#x4E86;&#x89E3; Google &#x5BF9;&#x4ED8;&#x8D39;&#x94FE;&#x63A5;&#x7684;&#x6001;&#x5EA6;</a>&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;</b>&#xFF1A;&#x5BF9;&#x4E8E;&#x8FD9;&#x4E9B;&#x7C7B;&#x578B;&#x7684;&#x94FE;&#x63A5;&#xFF0C;
          <a
          href="https://webmasters.googleblog.com/2019/09/evolving-nofollow-new-ways-to-identify.html">&#x4EE5;&#x524D;&#x63A8;&#x8350;</a>&#x4F7F;&#x7528; nofollow &#x5C5E;&#x6027;&#xFF0C;&#x73B0;&#x5728;&#xFF0C;&#x60A8;&#x4ECD;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x8BE5;&#x5C5E;&#x6027;&#x8FDB;&#x884C;&#x6807;&#x8BB0;&#xFF0C;&#x4F46;&#x66F4;&#x5EFA;&#x8BAE;&#x60A8;&#x4F7F;&#x7528;
            sponsored &#x6807;&#x8BB0;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rel=&quot;ugc&quot;</code>
      </td>
      <td style="text-align:left">
        <p>&#x5EFA;&#x8BAE;&#x5C06;&#x7528;&#x6237;&#x751F;&#x6210;&#x7684;&#x5185;&#x5BB9;&#xFF08;&#x4F8B;&#x5982;&#x8BC4;&#x8BBA;&#x548C;&#x8BBA;&#x575B;&#x5E16;&#x5B50;&#xFF09;&#x7684;&#x94FE;&#x63A5;&#x6807;&#x8BB0;&#x4E3A; <code>ugc</code>&#x3002;</p>
        <p>&#x5982;&#x679C;&#x60A8;&#x60F3;&#x5BF9;&#x503C;&#x5F97;&#x4FE1;&#x8D56;&#x7684;&#x8D21;&#x732E;&#x8005;&#xFF08;&#x59CB;&#x7EC8;&#x5982;&#x4E00;&#x5730;&#x505A;&#x51FA;&#x9AD8;&#x8D28;&#x91CF;&#x8D21;&#x732E;&#x7684;&#x6210;&#x5458;&#x6216;&#x7528;&#x6237;&#xFF09;&#x8868;&#x793A;&#x8BA4;&#x53EF;&#x548C;&#x5956;&#x52B1;&#xFF0C;&#x5219;&#x53EF;&#x4ECE;&#x4ED6;&#x4EEC;&#x53D1;&#x5E03;&#x7684;&#x94FE;&#x63A5;&#x4E2D;&#x79FB;&#x9664;&#x6B64;&#x5C5E;&#x6027;&#x3002;
          <a
          href="https://support.google.com/webmasters/answer/81749">&#x8BE6;&#x7EC6;&#x4E86;&#x89E3;&#x5982;&#x4F55;&#x9632;&#x8303;&#x5783;&#x573E;&#x8BC4;&#x8BBA;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rel=&quot;nofollow&quot;</code>
      </td>
      <td style="text-align:left">&#x5982;&#x679C;&#x5176;&#x4ED6;&#x503C;&#x4E0D;&#x9002;&#x7528;&#xFF0C;&#x5E76;&#x4E14;&#x60A8;&#x5E0C;&#x671B;
        Google &#x4E0D;&#x8DDF;&#x8E2A;&#x60A8;&#x7F51;&#x7AD9;&#x4E0A;&#x7684;&#x51FA;&#x7AD9;&#x94FE;&#x63A5;&#xFF08;&#x4E0D;&#x4ECE;&#x60A8;&#x7684;&#x7F51;&#x7AD9;&#x4E0A;&#x6293;&#x53D6;&#x7AD9;&#x5916;&#x94FE;&#x63A5;&#x9875;&#xFF09;&#xFF0C;&#x8BF7;&#x4F7F;&#x7528; <code>nofollow</code> &#x503C;&#x3002;&#xFF08;&#x5BF9;&#x4E8E;&#x60A8;&#x81EA;&#x5DF1;&#x7F51;&#x7AD9;&#x4E2D;&#x7684;&#x7AD9;&#x5185;&#x94FE;&#x63A5;&#xFF0C;&#x8BF7;&#x4F7F;&#x7528;
        robots.txt&#xFF0C;&#x5982;&#x4E0B;&#x6240;&#x8FF0;&#x3002;&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left"><em>&#x591A;&#x4E2A;&#x503C;</em>
      </td>
      <td style="text-align:left">
        <p>&#x60A8;&#x53EF;&#x4EE5;&#x5C06;&#x591A;&#x4E2A; <code>rel</code> &#x503C;&#x5408;&#x5E76;&#x4E3A;&#x4E00;&#x4E2A;&#x4EE5;&#x7A7A;&#x683C;&#x6216;&#x82F1;&#x6587;&#x9017;&#x53F7;&#x5206;&#x9694;&#x7684;&#x5217;&#x8868;&#x3002;<b>&#x793A;&#x4F8B;</b>&#xFF1A;</p>
        <ul>
          <li>&#x6211;&#x559C;&#x6B22;<code>&lt;a href=&quot;https://cheese.example.com/Appenzeller_cheese&quot; rel=&quot;ugc nofollow&quot;&gt;</code>&#x963F;&#x5F6D;&#x7B56;&#x5C14;<code>&lt;/a&gt;</code>&#x829D;&#x58EB;&#x3002;</li>
          <li>&#x6211;&#x8BA8;&#x538C;<code>&lt;a href=&quot;https://cheese.example.com//blue_cheese&quot; rel=&quot;ugc,nofollow&quot;&gt;</code>&#x84DD;&#x7EB9;<code>&lt;/a&gt;</code>&#x829D;&#x58EB;&#x3002;</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

Google 通常不会跟踪标有这些 `rel` 属性的链接。请注意，链接页也可能经由其他途径找到（例如站点地图或其他网站的出站链接），因此仍有可能被抓取。这些 `rel` 属性仅能在 `<a>` 标记中使用（因为 [Google 仅能跟踪 `<a>` 标记所指向的链接](https://support.google.com/webmasters/answer/9112205)），但 `nofollow` 属性除外，该属性还可以用作[漫游器元标记](https://support.google.com/webmasters/answer/79812)。

如果您不想让 Google 跟踪指向您的站内网页的链接，请使用 [robots.txt Disallow 规则](https://support.google.com/webmasters/answer/6062596)。

如果您不想让 Google 将某个网页编入索引，请允许抓取并使用 [noindex robots 规则](https://support.google.com/webmasters/answer/93710)。  


