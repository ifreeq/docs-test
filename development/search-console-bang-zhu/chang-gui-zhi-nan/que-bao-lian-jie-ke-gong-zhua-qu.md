# 确保链接可供抓取

只有当您的链接**使用正确的 `<a>` 标记**和**可解析的网址**时，Google 才能跟踪这些链接：

### 使用正确的 &lt;a&gt; 标记

只有当链接是含有 href 属性的 `<a>` 标记时，Google 才能跟踪这些链接。Google 的抓取工具不会跟踪采用其他格式的链接。Google 无法跟踪缺少 href 标记的 `<a>` 链接，也无法跟踪其他因为脚本事件而导致标记执行为链接时出问题的 &lt;a&gt; 链接。以下是 Google 可以跟踪以及无法跟踪的链接示例：

**可以跟踪的链接示例：**

* &lt;a href="https://example.com"&gt;
* &lt;a href="/relative/path/file"&gt;

**无法跟踪的链接示例：**

* &lt;a routerLink="some/path"&gt;
* &lt;span href="https://example.com"&gt;
* &lt;a onclick="goto\('https://example.com'\)"&gt;

### 指向可解析网址的链接

确保 `<a>` 标记所链接到的网址是 Googlebot 可以向其发送请求的实际网址，例如：

**可以解析：**

* https://example.com/stuff
* /products
* /products.php?id=123

**无法解析：**

* javascript:goTo\('products'\)
* javascript:window.location.href='/products'
* \#

