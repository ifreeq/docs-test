# 保持简单的网址结构

## 保持简单的网址结构

网站的网址结构应尽可能简单些。建议您组织一下您的内容，让网址的结构合乎逻辑并易于人们理解（尽可能采用易读的字词而非冗长的ID编号）。例如，如果您要搜索有关aviation的信息，http://en.wikipedia.org/wiki/Aviation一类的网址将可帮助您决定是否点击该链接。而 http://www.example.com/index.php?id\_sezione=360&sid=3a5ebc944f41daa6f849f730f1 一类的网址对用户的吸引力大大降低。

考虑在网址中使用标点符号。对于我们而言，**http://www.example.com/green-dress.html**这一网址比**http://www.example.com/greendress.html**有用得多。我们建议在网址中使用连字符\(-\)而非下划线\(\_\)。

过于复杂的网址，特别是那些包含多个参数的网址，可能会给抓取工具带来麻烦，因为它们可能会产生大量不必要的网址，全都指向您网站上相同或相似的内容。Googlebot可能会因此而消耗大量不必要的带宽，也可能无法将您网站上的所有内容完整编入索引。**此问题的常见原因**

导致网址过多可能有多种原因，其中包括：

* **对一组项目的过度过滤** 很多网站为同一组项或搜索结果提供不同的视图，通常可让用户使用定义的标准进行过滤（例如：显示海景酒店）。当以累加方式合并过滤器时（例如：带健身中心的海景酒店），网站中网址（数据视图）的数量就会急剧增加。因为Googlebot只需查看少量能用来访问各个酒店网页的列表即可，所以没有必要创建大量区别不大的酒店列表。例如：
  * 特价酒店：

    ```text
    http://www.example.com/hotel-search-results.jsp?Ne=292&N=461
    ```

  * 特价海景酒店：

    ```text
    http://www.example.com/hotel-search-results.jsp?Ne=292&N=461+4294967240
    ```

  * 带健身中心的特价海景酒店：

    ```text
    http://www.example.com/hotel-search-results.jsp?Ne=292&N=461+4294967240+4294967270
    ```
* * **动态生成文档**。由于计数器、时间戳或广告影响，这可能会产生少量变化。
* * **网址中的问题参数**。例如，会话ID会创建大量重复内容以及较多网址。
* * **排序参数**。 某些大型购物网站会提供多种方式来为相同的商品排序，从而造成网址数量大增。例如：

  ```text
  http://www.example.com/results?search_type=search_videos&search_query=tpb&search_sort=relevance
     &search_category=25
  ```

* **网址中不相关的参数，例如引荐参数**。 例如：

  ```text
  http://www.example.com/search/noheaders?click=6EE2BF1AF6A3D705D5561B7C3564D9C2&clickPage=
     OPD+Product+Page&cat=79
  ```

  ```text
  http://www.example.com/discuss/showthread.php?referrerid=249406&threadid=535913
  ```

  ```text
  http://www.example.com/products/products.asp?N=200063&Ne=500955&ref=foo%2Cbar&Cn=Accessories.
  ```

* * **日历问题**。 动态生成的日历可能会生成指向未来及过去日期的链接，而这些日期没有开始或结束期限。例如：

  ```text
  http://www.example.com/calendar.php?d=13&m=8&y=2011
  ```

  ```text
  http://www.example.com/calendar/cgi?2008&month=jan
  ```

* * **相对链接损坏** 。损坏的相对链接往往会导致无限循环。这个问题通常是由路径元素重复造成的。例如：

  ```text
  http://www.example.com/index.shtml/discuss/category/school/061121/html/interview/
    category/health/070223/html/category/business/070302/html/category/community/070413/html/FAQ.htm
  ```

**解决此问题的方法**

为避免网址结构出现潜在问题，建议您采取以下措施：

* 您可考虑使用robots.txt文件阻止Googlebot访问有问题的网址。一般情况下您应考虑阻止动态网址，例如会生成搜索结果或无限循环（如日历）的网址。在robots.txt文件中使用正则表达式可以轻松拦截数量较大的网址。
* * 尽可能避免在网址中使用会话ID。您可考虑改用Cookie。请参阅我们的[网站站长指南](https://support.google.com/webmasters/answer/35769)，以了解更多信息。
* * 截掉不必要的参数，尽量缩短网址。
* * 如果您的网站日历未设置期限，请为指向动态创建的未来日历页的链接添加[nofollow](https://support.google.com/webmasters/answer/96569)属性。
* * 检查网站是否存在损坏的相对链接。

