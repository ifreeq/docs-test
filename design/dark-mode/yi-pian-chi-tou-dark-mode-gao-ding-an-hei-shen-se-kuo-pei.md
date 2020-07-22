# 一篇吃透 Dark Mode ，搞定“暗黑/深色”适配

#### **下面分四块讲述：**

**1.两大平台做法：层级、颜色2.技术实现范围3.Dark细节4.Dark难点**文字如有思虑不周、描述不清之处，还望各位大佬指正。

#### **一、两大平台做法**

今年Android Q 和 iOS 13 都开始支持暗黑模式。下面我将从“层级”、“颜色”两方面讲述两大平台的做法。

#### **1.1 层级**

通过背景、投影，表达层级关系是设计常用的手法。如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/fd67e6d72b1bc9c2604490050b594679.jpg)

 列表在背景上方，所以背景颜色比列表暗。同理，搜索框颜色比背景色暗，可以理解为搜索框在背景的下层（凹槽）。可是在黑暗模式下，颜色明暗关系发生了变化：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/c7bc11aa5c169ba434c8df889b64f66a.jpg)

 iOS 13中的搜索框却比背景色要亮，二者的层级关系随之发生变化。PS：iOS 13中Sheet弹窗也会改变页面的层级关系。再看Android Q的设置页：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/464758bfa02a8b654dd245a290d71a02.jpg)

 在浅色模式下，页面背景为白色，搜索框也是白色，但通过投影表示，悬浮在背景上方。而两个提示卡片则采用灰色，表达的层级高低依次是：搜索框、背景、提示卡片

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/4052a72863d3b60d9d4974420ba1121a.jpg)

 在暗黑模式下，搜索框、背景、提示卡片 三者的明暗强弱依次是：搜索框、背景、提示卡片。透露出的层级关系与浅色模式下一致。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/70a80220c285b3a87584654ebd87a269.jpg)

 诚然iOS 13的暗黑适配效果，看起来比Android Q优秀。毕竟苹果每个元素都经过精心设计：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/c6cc5a56b761c1f5d7ae37e53b34ea04.jpg)

 但层级变化上，缺少统一的“标尺”，这无疑会增加多人协作的适配成本。而安卓则有一套层级体系可以借鉴。

无论是浅色模式，还是暗黑模式，只要固定好各层级，再确定不同层级的配色方案，就能大大降低适配难度。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/4ca442cf9834f19e728599819fde24fc.jpg)

#### **1.2 颜色**

说完“层级”再来看下“颜色”。这里先说与“层级”关联度较大的“黑白灰”，再说“彩色”。

 **① 灰色**iOS 13这边，以新版的健康APP为例：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/bf4f7ff9c69a3a17b35828be83bc4059.jpg)

 左图是健康APP的首页，右图是点击“编辑”后调起的弹窗。在这两个看起来很简单的页面中，排除蓝色、红色和所有文字颜色，还用到8种颜色，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/fd145bc81b849ffd53f80ef2a047a779.jpg)

 而苹果官方给出的配色建议，则更加“丰富”：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/e3724755c8d469aa9dce23462f28c0e9.jpg)

 不仅明暗不同，且一部分是透明色。另外，这些颜色的色值变化也没有明显的“规律”可循。如果按照iOS 13的规范进行操作的话，不仅执行成本高的吓人，统一性也很难得到保障。相比之下，安卓所采用叠加“不透明度”不同的白色，来区分层级的方式执行起来就简单很多。在Android Q中，安卓官方的建议是：在浅色模式下，页面的层级可以靠投影进行区分，底层元素投影面积较小、而上层元素投影面积较大。如下图左侧：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/b937689dd51738bf56fdf17d946c439d.gif)

 而在暗黑模式下投影变得不明显。因此改用“明暗”区分层级，如上图右侧。Android Q把页面分层了8个层级：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/e4a7d3fa237f724be39dfa5af5ebeeee.jpg)

 其中最底层的”代号”为00dp\*，最高层为24dp。PS：这里的dp，无需和逻辑像素单位dp、pt关联。不同层级的实现方式，是在同一个背景色上，叠加“不透明度”不同的白色。

比如：背景色为\#121212， 那么最底层的00dp就是\#121212叠加不透明度为0%的白色；而最上层的24dp，则是\#121212，叠加不透明度为16%的白色。这样一来不同层级只要调整所叠加的白色不透明度即可，操作起来也比较方便。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/d160e9640efa15ef3cc1a121478d806e.jpg)

 **② 彩色**下面再看彩色，先说iOS 13中的彩色方案

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/54038ee6067fafe713556e23197167d1.jpg)

 如图所示，两种模式下的颜色变化很小。RGB的色值不太方便看差异，下面我将RGB配色改为HSB配色。如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/340430f0d40129ed64bb17218646146c.jpg)

 可以看到在HSB模式下，参数只有2~5的细微差别。作为凡人的我，只得感叹苹果配色的微妙和不易琢磨…..相比iOS“微妙精致”配色，安卓的彩色适配方案则要简单粗暴很多。过去安卓的配色规范中，一直有一块彩色梯度渐变。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/30718899b55adfb939c996d191267207.jpg)

 通常浅色模式的配色会选取500左右的配色为主色，如下：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/887c5c8b7902ed1dde9b19eeeb4a0e11.png)

 而这个配色方案在暗黑模式下，与背景色的对比度太小，

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/7ccd333534a191804669f55f4b7ff94a.png)

 所以官方不建议在暗黑模式下使用与浅色模式相同的主色。对此，安卓官方给的建议是，在渐变梯度上，选用200左右的颜色作为暗黑模式下的主色。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/dd40bf15b1b6e3c621bc19aa118a90a0.png)

 更改的页面如下:

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/aca5780b5ddd42d51ad346664d81264e.png)

小结：安卓彩色适配看起来虽没有iOS精致，但终归是有迹可循，值得借鉴。

#### **二、标注的变化**

近期更新的爱奇艺 iOS V10.8.5新增了暗黑模式适配

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/dfcc407f71b6643913300a7a27164abe.jpg)

 根据前端开发工程师描述，目前Native、Html 5 均支持暗黑模式适配（跟随系统切换）

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/d1c6275fce2b288460298bb0b68e0034.png)

如此同时，对于拥有**浅色**和**深色**两种模式的项目（抖音APP只有深色），今后设计师在提供颜色标注时，也需要提供两套色值。以按钮色值标注为例：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/b3aa304f85f82b7bdbfa3206d5320a75.png)

 假如一个按钮浅色模式模式下使用APP主色\#10C30B，在暗黑模式下，主色为\#0FD852。那过去我们只需要在对应文字旁，标注\#333333即可，如下图

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/2aa4ad1ced2064de6ccd182293fab237.png)

 而新版的标注，不仅要标注\#333333，还要标注\#CCCCCC，如下图

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/d35ad0e5e62db2291a7c70b765f67ad9.png)

 日常需求，可以不输出暗黑模式下的设计图，仅提供两种模式下的标注即可。但切图就需要提供双份：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/009d2be1970c29bcf65fca7934fb26cd.png)

一份浅色模式下的@2x、@3x切图，一份暗黑模式下的@2x、@3x切图。

#### **三、四个适配细节**

下面分别讲述Dark适配需要关注四个细节：字色、图标、插图、其他

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/001558c83357d683d847715b0aa5826c.png)

#### **3.1 字色适配**

过去常见的文字配色为\#333333、\#666666、\#999999、\#CCCCCC

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/f4a12237530a4c0d0286642aae3b1823.png)

 这些字色都是不带透明度的色值。这四种色值在HSB的模式下，H和S相同，只在B上存在差别。而亮度差，刚好构成了字色上的“层级”。PS：详见之前写的[《深色模式文字配色》](http://www.xueui.cn/tutorials/color-tutorial/diablo-mode-text-colour-matching.html)为了适配暗黑模式，建议字色采用某个固定色值，字色的层级仅靠透明度进行区分，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/0571a44e2d7e295c0e155589a7f7b8ec.jpg)

 这样做，不仅方便后期标注。也能让文字和背景之间保持良好的对比度。如果采用不带透明度的文字，那暗黑模式下，很容易出现文字和背景之间对比度较低，严重的会导致文字和背景“相融”，对阅造成障碍，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/42b09aa23693ab0d7b3ca70dd93861b5.png)

 而采用带透明度的配色方案，可以让文字和背景之间保持一个相对好的对比度：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/044dcde86470d818bc451c1d36c06c15.png)

避免出现文字和背景相融的Bug

#### **3.2 图标适配**

下面是苹果原生键盘中的Emoji Key按钮。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/1f43f698c27b956f7f0ddf219672b21c.jpg)

 在iOS 9上，苹果公司采用的是办法将浅色模式下的线性图标颜色反转，拿到深色界面下使用。等到了iOS 11，苹果公司将这个图标在暗黑模式下改成了面性图标。这是因为白色背景可以更好的表现出形状，人的大脑可以将白色脑补成图形的一部分。然而在暗黑模式下，这种作用消失了，人脑更倾向于认为这些空白的部分是镂空的。比如电影哪吒魔童结尾的彩蛋图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/4284b6fee57e7c307eb2c39de551c868.jpg)

 看到黑底白线的卡通人物，会不会觉得怪怪的？尤其是眼睛👀下面将彩蛋哪吒反色处理后，变成白底黑线，再看下效果：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/2d7f416420bf73160e49b0f29b896c2c.jpg)

 是否觉得白底黑线，更有立体感？所以苹果在iOS 13暗黑模式下，放弃线性图标，改用面性图标。比如Emoji键盘上的图形：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/7d5126c8138f7de97e0091c06913903f.jpg)

 不过键盘最右侧的“删除”，在暗黑模式下却依然采用线性图标。我想删除按钮用线性图案，也许是苹果刻意为之。毕竟在深色键盘中，诸如地球、麦克风这个两个图形，也都是线性图标。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/43238f7a6cb7cfb0b3f3c2a538ab028e.jpg)

 也就是说，暗黑模式下采用面性图标并非一招鲜的解决办法。最终还是需要设计师根据具体效果进行选择。在爱奇艺 iOS V10.8.5 中，底部导航在暗黑模式下，也由线性图标变成了面性图标。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/e9a236aead700e0dcef463a393d88946.jpg)

 值得一提的是，有一些浅色模式下“面性图标”（填充图标），到暗黑模式下，也是要进行再设计的。比如下面的页面：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/061960b1088906a1db5304d1dd3fa75c.jpg)

 灰色图标和彩色图标，在浅色模式下，图案比圆形的底色要深。这样的设计在暗黑模式下，效果并不理想。如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/49fdc1e859c6bd2d5624ddeab72abd14.png)

 因此需要对图标的图形和圆形底色进行调整。改为底色暗、图形亮。如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/f7e287011f1e22ea0ea25c329a3f017d.png)

 修改前后对比如下：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/ea64345115d6e3a236637f6944550980.png)

#### **多说一点：**

有些文章中提到，在适配暗黑模式时，可以提供SVG格式的切图。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/9a261eae12ec276445c769dac92bc6e0.png)

 这样就可以让程序员调整切图颜色，减少设计师工作量。在我看来，SVG切图完全不符合国情。首先，国内的APP产品，很少使用单色图标，以“多色”、“彩色”、“带投影”为主，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/ca06d8a69927e6e6baa2061e6f2a97c9.jpg)

其次，如果只是局部使用SVG，那后期维护成本不低。远不如png切图维护起来方便。还有就是，矢量切图在@3x下，不可避免的会出现像素不对齐、描边线虚边这类Bug。简单来说，SVG虽有亮点，但缺点尤多，慎用。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/185379ca65b84f6ccbc88d802fb0ccac.jpg)

#### **3.3 插图适配**

现在大家都说暗黑模式更加适合夜晚操作，那想必大家都不希望在操作一款适配暗黑模式的APP时，时不时出现“亮瞎”眼睛的白色页面。普通的页面适配起来比较简单，程序员就可以完成适配。但一些带有插图的页面，就免不了需要设计师专门进行适配。下面这些页面，就需要设计师注意了：

 **① 启动页**以下样式，logo中带有黑色、图中有很亮的配色（比如黄色），需重绘：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/cb9a1b379d25859795343a779685d966.jpg)

 下面这种本来就是黑色，以及大面积品牌色的做法，可直接复用：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/be7d5c3dc5c57e0d2871f2189b688b9c.jpg)

 **② 新手引导**原本适配白底的插图，快重绘制吧：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/026443c296a37500c17ada68bdb82e0b.jpg)

 彩色底、品牌用色、大面积彩绘的方案，可直接复用：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/56d23874ba3a22a4101ca393ac3366f2.jpg)

 **③ 弹窗**暗黑模式下，弹窗颜色也有调整，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/f2932c625672e12211bb3c47e3e68e11.jpg)

 出现在白底弹窗上的配图，难免要重绘，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/dca7b0127f92edf246ac058e5025e697.jpg)

异形弹窗可直接复用，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/1b2bb82a6aeb4a108b82e9fe6431d39e.jpg)

 **④ Loading**以蚂蜂窝APP为例，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/edb2ae8dc54b642d32bb2dbb9b0854cf.gif)

 蚂蜂窝logo中存在黑色，且logo动画中用到了浅灰色。

那么如果这个动效要适配深色底的背景，无疑需要重绘。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/9a8905c46b5de791cff9b2aa6d541a54.png)

 **⑤ 空态插图**

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/0158a8d8ab1534187c3234b9b796879b.jpg)

 线性插图、白色插图、黄色插图（亮色插图）也需要重绘。其中带眼睛的线性插图，无论是反色、还是简单的转成面，在深色上看起来都会很奇怪。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/bb16b58905526a76f4e82d2627dcabe4.jpg)

 这个时候，建议参考爱奇艺Mac端的插图方案：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/f5fbe634e99f46d822810e36fa1ff8e1.jpg)

保留眼白和眼球，将整个图形调暗。

 **⑥ 其他**还有一些，诸如身高体重设置、性别选择、内容推荐、勋章展示等页面，其中的元素和插图

在暗黑模式下，难免也需要重新绘制。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/3152185ce156b2a3d5e1d581ab46cbd0.jpg)

#### **3.4 元素适配**

在APP中，有一些元素不像图标和插图那么明显，导致经常被忽视。但其实又经常能够遇见。

 **① 默认图和默认头像**

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/c883f0c85e12bb3c933bb9dcb5239d9d.jpg)

 在进行暗黑模式适配的时候，如果忽略这些元素，整个产品的体验会大打折扣。在爱奇艺 iOS V10.8.5 中，首页默认图已适配Dark模式。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/d56bbecdae75eaa183e5ec4a7a699317.jpg)

 但其他模块的视频预加载尚待适配，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/34afc9c52da1943c7b9cbea5db8d61e7.jpg)

 **② 标签**还有一些标签图，需要进行二次设计：

比如爱奇艺首页的“剧头条”的插图和“最新”标签。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/6425bd1d92762f29f1d5155e532a6df0.jpg)

 **③ Logo**iOS 13邮箱设置界面中，雅虎logo为深紫色，Aol.为黑色。在暗黑模式下，苹果对这两个logo的颜色都做了改变。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/e61b37ebd857ab23f2bba3399e14162b.jpg)

 **④ 地图**最后，还有一个工作量巨大的模块：地图iOS 13中，苹果地图已经完成了暗黑模式的适配。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/0fe188ac9998e8218362c767ab8d9a65.jpg)

 这样一来，夜间开车看地图再也不用担心手机晃眼了。🚘🤦但对于一些公司来说，地图适配恐怕要头疼了。比如自如、美团、微信：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/798bbbbc82e0bc37a93da9435bb73808.jpg)

 如果地图要自家设计适配的话，这工作量恐怕得要个一年半载。即便提供地图服务的公司来完成地图暗黑模式的适配，这些公司在地图上用到的小元素，自然还需要自家设计师来处理。

#### **四、Dark适配难点**

#### **4.1 重绘**

除了上面提到的各种图标、插图、小元素 重绘外

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/bc3d3592b88b174cc46b3bec2ef95bc4.jpg)

 还有一些不起眼的H5页面需要进行适配，比如：用户协议、加载页、功能介绍，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/78784191c313aa6d30312c55ed7d0ad2.jpg)

 目前豆瓣APP也做了Dark模式适配，完善度相当高。就连Tost、加载中这样的细节都做了适配，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/52204ccde9939f8acbc5d08081901e7e.jpg)

 不过，当下豆瓣APP（V6.22.0）分享弹窗图标尚待完善，如下方左图：

右侧放了爱奇艺的弹窗图标，方便做个对比。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/aadfcfe5a56bfc5166b6763bf9d588d3.jpg)

#### **4.2 对比度**

无论是Android Q，还是iOS 13，在暗黑模式的建议中，都提到了文字需满足WCAG无障碍阅读的AA标准。WCAG从高到低分别是AAA、AA、A三个等级。其中AA要求文字与背景对比度不小于4.5:1

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/1dbad657702baeb3e837385cd332f7e7.png)

 不过，WCAG还规定，大号文字与背景的对比度大于3:1就算符合要求。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/a5ac9a58dc161be5b14e9edce55aaeb1.png)

 同理，小字号要求的对比度会高于4.5:1 ，加之国内正文字号普遍小于18dp（@2x，36px）所以单纯比较对比度，并不实用。比如，现在很多人推荐的 contrast-ratio.com （在线测量对比度）：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/99ae0a2857ebca659c5fea66caca74a3.jpg)

 这个网站只能提供两个色值的对比度，功能十分局限。建议使用hexnaw.com进行对比度测量：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/f604db18336479176036a57827a6ac20.jpg)

 这个网站支持最多12种颜色的交叉对比：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/09c392c22d71e1b4d18131feaa07bee9.png)

 这里输入3种色值，就能看到交叉对比的结果：

第一个参数是对比度，后面分别对应的是大字号、小字号的WCAG评级（AAA最高，AA普通，A最低，Naw不及格） ****

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/31ad62ed0b7ab60bc538bfd4a46753b4.jpg)

#### **4.3 遗漏**

要完成产品的暗黑模式适配，最大的难题就是遗漏。以爱奇艺 iOS V10.8.5为例部分主页完成了暗黑模式适配，但很多二级页面尚待适配，如下图：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/c40fd905908c252345e4500b935146b6.jpg)

 另外，一些页面虽然“大面”上完成了适配，但如果细节处理不完善，效果就会大打折扣，比如爱奇艺应援页面：

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/eb89fcf5d6a933fd2ec1bb1c7a360cd1.jpg)

 仅这一页，就尚有四处有待进一步完善。现在的效果，无疑破坏了产品在暗黑模式下体验的一致性。对于容易出现的遗漏，建议如下：首先在适配前，用Excel统计好需要适配的产品页面。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/30ceda57fdc8954f911cb1a84a284aa0.png)

 其次，标记好各种不常见的页面模块和组件，比如低频的运营配置模块。同时在替换素材和切图时，避免采用“替换全局”这种不便后期排查的描述。而是标记好需要替换的页面，并在开发后逐一排查。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/c8ea3a30dbd68de92054a7bfe5e8bae7.jpg)

 如果一味把工作量和责任推给开发，也成。不过，当别人看到不完善的页面时，一定不会觉得是程序员的锅，而只会吐槽设计师。

#### **五、最后**

如果你还没有开始适配Dark，那我希望你在看完本文后，可以对适配工作量有一个较为全面的了解。重新思考是否要进行Dark适配，不要走过去“MD风格适配”的老路。Dark适配一旦开工，就不好轻易掉头和叫停。也许眼下，觉得做暗黑模式很新鲜，可时间一长，多数需求都要做两套，那可就是另一副光景了。

![](http://imgs.xueui.cn/wp-content/uploads/2019/09/496790cc642ab51ea35c1ebede88f4e9.jpg)

### 转载声明

本文转载自: [http://www.xueui.cn/design-theory/dark-dark-adaptation.html](http://www.xueui.cn/design-theory/dark-dark-adaptation.html)

尊重原创, 转载请注明出处

