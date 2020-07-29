# 实用资源：面向适合在移动设备上显示的网页的开发者



现在，用户通过移动设备浏览网站的频率与通过桌面设备浏览网站的频率相当（或比后者更高）。所以，您应将自己的网站设计为可通过移动设备浏览。

移动设备通常分为两大类：智能手机和功能手机。智能手机在本质上是配有小屏幕的小型桌面设备，包括 Android 和 iPhone 设备（出于设计方面的考虑，我们将平板电脑也列入此类）。功能手机是所含功能较少的移动设备。Google 会分别对这两类手机采取不同的处理方式。

**以下几点均是针对这两类设备而言：**

* Google 搜索结果页应适合在发出请求的那类手机（功能手机或智能手机）上显示。
* 您可以使用 [App Indexing Search Preview](https://firebase.google.com/docs/app-indexing/android/test#preview-search-results-on-android) 来查看该页在此类手机上的显示效果。
* 您可以在 robots.txt 文件中屏蔽此类移动设备（智能手机或功能手机）专用的用户代理。
* [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) 可以分析网页，并针对如何缩短网页加载时间提供建议。
* [详细了解如何制作适合在移动设备上显示的网站](https://developers.google.com/search/mobile-sites/)。

### 实用资源和信息：面向适合在智能手机上显示的网站的开发者

* 在部分国家/地区，如果检测到网络连接速度较慢，Google 会自动对搜索结果所链接到的网页进行转码。您可以使用[低带宽转码器测试工具](https://www.google.com/webmasters/tools/transcoder)查看相应网页的显示效果。[了解详情](https://support.google.com/webmasters/answer/6211428)。
* [移动设备适合性测试](https://search.google.com/test/mobile-friendly)描述了可能会在用户通过智能手机访问您的网页时对该网页产生影响的问题。
* Search Console 的[“在移动设备上的易用性”报告](https://www.google.com/webmasters/tools/mobile-usability)描述了可能会在用户通过智能手机访问您的网页时对该网页产生影响的问题。
* 在特定的国家/地区，Google 会在网络连接速度较慢时重新调整 SERP 中图片的大小，除非相应的网页所有者已[明确选择停用](https://support.google.com/webmasters/answer/6149573)这一功能。
* Chrome 浏览器可在[设备模式和移动设备模拟器](https://developers.google.com/web/tools/chrome-devtools/device-mode)中支持多种智能手机版式。

### 实用资源和信息：面向适合在功能手机上显示的网站的开发者

* 无论网络连接速度如何，Google 都会在全局范围内对 SERP 以及由 SERP 链接到的所有网页进行转码，除非：
  * 该网页已“适合在功能手机上显示”； 或
  * 该网页包含 &lt;link rel="alternate" media="handheld" href="alternate\_page.htm" /&gt; 标记。
* Google 会重新调整 SERP 中以及由 SERP 链接到的任何网页中所有图片的大小，除非用户已明确选择停用这一功能。
* [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) 

**适用于功能手机的标记**

适合在功能手机上显示的网页可以采用多种标记语言，包括 WML、XHTML Basic、XHTML MP 和 cHTML。您应根据自己的目标市场做出适当选择。

**适用于功能手机的标记标准：**

* [cHTML](https://zh.wikipedia.org/wiki/C-HTML)
* [WML 1.3](http://en.wikipedia.org/wiki/Wireless_Markup_Language)

**适用于功能手机的验证程序：**

* [Mobile-friendly XHTML Validator](http://validator.w3.org/mobile/) \(W3C\)
* [Mobile-readiness checker](http://ready.mobi/) \(.mobi\)

**适用于功能手机的模拟器**

* [i-mode emulator](http://www.google.com/search?q=i-mode+emulator) \(DoCoMo\)
* [User-Agent Switcher](https://addons.mozilla.org/en-US/firefox/search/?q=user-agent%20switchers)（Firefox 插件）

