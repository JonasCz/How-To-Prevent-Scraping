# Web 防爬指南
**（或至少让其更难抓取）**

---

提示：这篇文章是我 Stack Overflow [这个问题](http://stackoverflow.com/a/34828465/4428462)回答的扩展，我把它整理在 Github 因为它实在是太长了，超过了 Stack Overflow 的字数限制（最多 3 万个字，这文章已经超过 4 万字）

欢迎大家修改、完善还有分享，本文使用 [CC-BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) 许可。

---

**本质上说，防抓的目的在于增加脚本或机器获取你网站内容的难度，而不要影响真实用户的使用或搜索引擎的收录**

不幸的是这挺难的，你需要在防抓和降低真实用户以及搜索引擎的可访问性之间做一下权衡。

为了防爬（也称为网页抓取、屏幕抓取、网站数据挖掘、网站收割或者网站数据获取），了解他们的工作原理很重要，这能防止他们能高效爬取，这个就是这篇文章的主要内容。


通常情况下，抓取程序的目的是为了获取你网站特定的信息，比如文章内容、搜索结果、产品详情还有网站上艺术家或者相册信息。他们爬取这些内容用于维护他们自己的网站（甚至通过你的内容赚钱！），或者制作和你网站不一样的前端界面（比如去做移动 APP），还有一些可能作为个人研究或分析使用。

实际上，有特别多的爬虫类型，而且他们的爬取方式都不太相同：

* 蜘蛛，比如 [Google's bot](http://googlebot.com/) 或者网站复制工具 [HTtrack](http://www.httrack.com/)，他们访问你的网站，而且在页面中寻找链接后递归的去爬取以获取页面的数据。有时候他们只用于获取特定数据，而不会全部爬取，通常结合 HTML 分析器在页面中获取想要的数据。

* Shell 脚本，有时候，通用的 Unix 工具也被用来爬取：wget 或者 curl 用于下载页面，用 Grep （Regex） 去分析获取想要的数据，一般都会写一个 Shell 脚本。这些都是最简单的爬虫，也是最脆弱的一类爬虫([Don't ever try parse HTML with regex !](https://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454#1732454)），所以是最早发现和防范的爬虫。

* HTML 爬取器和分析器，会基于 [Jsoup](http://jsoup.org/)、[Scrapy](http://scrapy.org/) 还有其他一些工具。和 shell 脚本基于正则类似，他们呢基于特定模式（pattern）分析你的 HTML 结构从而获取想要的数据。

  举个例子：如果你的网站有搜索功能，那这个爬虫可能会模拟提交一个搜索的 HTTP 请求，然后从搜索结果页中获取所有的链接和标题，有时候会构造有成千上百不同的请求，只是为了获取标题和链接而已。这是最常见的一类爬虫。
  
* 屏幕爬取，比如会基于 [Selenium](http://www.seleniumhq.org/) 或者 [PhantomJS](http://phantomjs.org/)，他们实际上会通过真实浏览器打开你的网站，因此也会运行你网站的 JavaScript、AJAX 或者其他的，然后他们从你的页面获取自己想要的文本内容，通常：
  
    * 等页面加载完毕， JavaScript 也执行之后，从浏览器获取 HTML 结构，然后用 HTML 分析器去获取想要的数据或者文本。这是常见做法，所以有很多方法可以防止这类 HTML 分析器或爬虫。
    * 获取加载完页面的屏幕截图，然后使用 OCR 分析从截屏中获取想要的数据。这类不唱将，而且只有非常想要你网站内容的爬虫才会使用。
  
  基于浏览器的爬去很难处理，他们运行脚本，渲染 HTML，就像真实用户一样访问你的网站。

* Web爬取服务，比如 [ScrapingHub](http://scrapinghub.com/) 或者 [Kimono](https://www.kimonolabs.com/)。实际上，很多人的工作就是找出如何爬取你的页面获取其中内容并给其他人使用。他们又是会用大量的代理甚至修改 IP 地址来绕过频率限制和封禁，所以他们反爬的关键对象。

  毫无疑问，防止专业的爬取服务是非常困难的，但是如果你增加爬取的难度，或者增加他们找出爬取方法的时间，那这些人（或者付钱让他们做的人）可能会放弃爬取你的网站。
  
  
* 把你的网站页面通过 [frames](https://en.wikipedia.org/wiki/Framing_(World_Wide_Web)) 嵌入其他站点，或者把你的页面嵌入移动 APP。
  
  虽然没有什么技术含量，但是确实也是个问题，比如移动 APP（Android 和 iOS）可以嵌入你的网站，甚至可以注入一些自定义的 CSS 和 JavaScript，所以可以完全改变你网站的外观，然后只展示想要的信息，比如只展示文章内容或者搜索结果，然后隐藏你网站的 headers、footers 还有广告。
  
* 人肉复制粘贴：人肉复制和粘贴你网站的内容到其他地方。很遗憾，这个没有好的方法加以防范。

以上不同的爬虫类型有很多相同点，很多爬虫的行为也很相似，即使他们使用了不同的技术或者方案去爬取你的内容。

这些大多数都是我自己的想法，我在写爬虫时也遇到许多困难，还有一些是来源于网络。

## 如何反爬

一些常见的检测和防止爬虫的方法：

### 监控你的日志和请求；当发现异常行为时限制访问

周期性的检查你的日志，如果发现有异常活动表明是自动爬取（爬虫），类似某一个相同的 IP 很多相同的行为，那你就可以阻止或限制访问。

一些常用的方法：

* **频率限制**
  
  只允许用户（或爬虫）在一段时间内访问特定的次数，举个例子，某个 IP 或用户只允许一分钟搜索很少的次数。这个会减慢爬虫的爬取速度，让他们变得低效。如果次数特别多或者比真实用户多很多，那你也可以显示验证码页面。
  
* **检测异常行为**

  如果你能看到异常行为，比如同一个 IP 有很多相同的请求，有些会翻很多页或者访问一些异常的页码，你可以拒绝访问或者在后续的请求中展示验证码。
  
* **不要只通过 IP 检测和限制，也要用其他的用户标识**

  如果你做了访问限制或频率限制，不要只简单的根据单个 IP 地址去做；你可以通过其他的标识和方法去识别一个用户或爬虫。一些可以帮你识别用户/爬虫的标识：
  
  * 用户填写表单的速度，还有他们点击按钮的位置
  * 你可以通过 JavaScript 获取很多信息，比如屏幕的大小 / 分辨率，时区，安装的字体等等，你可以用这些去识别用户
  * 携带的 HTTP 头，特别是 User-Agent
  
  举个例子，如果某个 IP 请求了你网站很多次，所有的访问都有相同的 UserAgent 、屏幕尺寸（JavaScript 检测），还有全都使用同样的方式和固定的时间间隔点击同一个按钮，那它大概是一个屏幕爬虫；你可以临时限制相似的请求（比如 只限制来自那个 IP 特定 User-Agent 和 屏幕尺寸的请求），这样你不会误伤同样使用这个 IP 的真实用户，比如共享网络链接的用户。
  
 更进一步，当你发现相似的请求，但是来自不同的 IP 地址，表明是分布式爬虫（使用僵尸网络或网络代理的爬虫）。如果你收到类似大量的请求，但是他们来自不同的 IP 地址，你可以拒绝访问。再强调一下，小心不经意限制了真实用户。
 
 这种方法对那种运行 JavaScript 的屏幕爬虫比较有效，因为你可以获得他们大量的信息。
 
 Stack Exchange 上关于 Secruity 的相关问题：
 
 * [How to uniquely identify users with the same external IP address? ](http://security.stackexchange.com/questions/81302/how-to-uniquely-identify-users-with-the-same-external-ip-address)

 * [Why do people use IP address bans when IP addresses often change? ](http://security.stackexchange.com/questions/96377/why-do-people-use-ip-address-bans-when-ip-addresses-often-change) 关于这个方法的更多限制

* **使用验证码，而不是临时限制访问**
  
  对于频率限制，最简单的方式就是临时限制访问，然而使用验证码会更好，看下面关于验证码的部分。
  
### 要求注册和登录

如果可行，要求创建用户才可以查看你的内容。这个可以很好的遏制爬虫，但很容易遏制真实用户：
 
 * 如果你要求必须创建账户和登录，你可以精准的跟踪用户和爬虫的行为。这样的话，你可以很简单就能检测到有爬取行为的账户，然后封禁它。像频率限制或检测滥用（比如段时间大量搜索）就变得简单，你也可以不仅仅通过 IP 去识别爬虫。

为了防止爬虫创建大量的用户，你应该：

 * 注册时需要提供 email 地址，发送一个验证链接到邮箱，而且这个链接必须被打开才可以激活这个账户。一个邮箱只允许一个账户使用。
 * 在注册或创建用户时，必须通过验证码验证，为了防止创建用户的自动脚本。

要求注册用户对用户和搜索引擎来说不友好；如果你要求必须注册才可以看文章，那用户也可能直接离开。

### 阻止来自云服务商和抓取服务的 IP

又是，爬虫会被运行在云服务商，比如 Amazon Web Services 或 Google App Engine，或者其他 VPS。限制这些来自云服务商的 IP 地址访问你的网站（或出验证码）。你可以可以直接对来自爬取服务的 IP 地址限制访问。

类似的，你也可以限制来自代理或 VPN 的 IP 地址，因为很多爬虫会使用它们来防止被检测到。

但是要知道限制来自代理服务器或 VPN 的 IP，也很容易影响到真实用户。

### 当你封禁时不要展示具体错误信息

如果你要封禁或限制访问，你应该确保不要把导致封禁的原因告诉爬虫，他们会根据提示修改自己的爬虫程序。所以下面的这些错误最好不要出现：

* 你的 IP 访问太多次了，请重试
* 错误，没有 User Agent

想法的，展示一些不包含原因的友好信息会更好，比如下面的信息会更好：

* 对不起，有些不对劲。如果问题持续出现，你可以联系 helpdesk@example.com 以获取支持。 
  
如果真实用户看到这个错误页面，这个对真实用户来说也十分友好。在后续访问中，你也可以考虑展示验证码而不是直接封禁，如果真实用户看到这个错误信息，对于合法用户也会联系你。

### 如果有爬虫访问，请使用验证码

验证码（Captcha，“Completely Automated Test to Tell Computers and Humans Apart”)对于防爬十分有效。不幸的是，它也很容易惹恼用户。

因此，如果你发现疑似爬虫，而且想要阻止它的爬取行为，而不是封禁它以免它是真实用户。你可以考虑显示验证码在你再次允许访问之前。

使用验证码的注意事项：

* 不要造轮子，使用类似 Google 的 [reCaptcha](https://www.google.com/recaptcha/intro/index.html) 的一些服务：比你自己实现一套验证码服务要简单太多，对于用户来说，也比识别模糊或扭曲的文字更友好（用户一般只需要点击一下），而且相对于你自己提供的简单验证码图片，爬虫也更难去破解
* 不要在 HTML 结构中包含验证码答案：我曾经看到过一个网站在它自己页面上有验证码的答案，（虽然隐藏的很好）所以让验证码根本就没有用。不要做类似的事情。再强调一次，用像 reCaptcha 的服务，你将不会有类似的问题（如果你正确地使用它）

### 将你的文字转位图片

你可以在服务端将文字渲染成图片显示，他将防止简单的爬虫去获取文字内容。

然而，这个对于屏幕阅读器、搜索引擎、性能还有一些其他一些事情都不太好。这个在某些方面也是不违法的（源于访问性问题，eg. the Americans with Disabilities Act），而且对于一些 OCR 爬虫也能非常简单的规避，所以不要采用这种方式。

你也可以用 CSS sprites 做类似的事情，但是也有相同的问题。

### 不要暴露你完整的数据

如果可以的话，不要让脚本/爬虫能获取你所有的数据。举个例子：你有一个新闻站，有大量的个人文章。你应该确保文章只能通过你自己网站的搜索到，如果你网站不是到处都有你的文章还有链接，那么确保它们只能被搜索功能访问到。这意味着如果一个脚本想要获取你网站的所有文章，就必须通过你站点文章中所有可能的短语去进行搜索，从而获取所有的文章列表，但是这个会非常耗时，而且低效，从而让爬虫放弃。

以下操作会让你完全暴露：

* 爬虫/脚本并不想/需要获取所有的数据
* 你站点文章的链接看起来是这种方式 `example.com/article.php?articleId=12345`，这样做（或类似的其他做法）会让爬虫很简单的迭代 `articleID` 就能获取你所有的文章
* 还有一些其他方式获取所有的文章，比如写一个脚本去递归的爬取你文章中其他文章的链接
* 搜索一些常用的词比如 "一" 或者 "的" 将会暴露几乎所有的内容。所以这点是要注意的（你可以通过只返回 top 10 或 top 20 来避免这个问题）
* 你的文章需要被搜索引擎收录

### 不要暴露你的 API、节点或其他类似的东西

确保你不会暴露你的任何 API，很多时候都是无意间暴露。举个例子，如果你正在使用 AJAX 或 Adobe Flash 的网络请求 或 Java Applet（千万不要用！）来加载你的数据，从这些网络请求中找到要请求的 API 十分简单，比如可以进行反编译然后在爬虫程序中使用这些接口。确保你混淆了你的 API 并且其他人要用它会非常难破解。

## 阻止 HTML 解析器和爬取器

由于 HTML 解析器是通过分析页面特定结构去获取内容，我们可以故意修改这些结构来阻止这类爬虫，甚至从根本上他们获取内容。下面大多数做法对其他类型爬虫如搜索引擎蜘蛛、屏幕爬虫也有效。

### 经常修改你的 HTML 结构

处理 HTML 的爬虫通过分析特定可识别的部分来处理 HTML。举个例子：如果你所有的页面都有 id 为 `article-content` 的 `div` 结构，然后里面有你的文章内容，那要获取你站点所有文章内容是十分简单的，只要解析那个 `article-content` 那个 `div` 就可以了，然后有这个结构的爬虫就可以把你的文章随便用在什么地方。

如果你常常修改 HTML 还有你页面的结构， 那这类爬虫就不能一直使用。

* 你可以经常修改你 HTML 的 `id` 和 `class`，甚至让他能自动改变。所以，如果你的 `div.article-content` 变成 `div.a4c36dda13eaf0`，而且每周都会变化，那么爬虫一开始可能能正常工作，但是一周之后就不能使用了。同时也确保修改你 id/class的长度，这样也可以避免爬虫使用类似 `div.[any-14-characters]` 来找到想要的 `div`。
* 如果无法从标记中找到所需的内容，则抓取工具将通过HTML的结构方式进行查找。所以，如果你的所有文章都有类似的结构，比如每个 `div`，并且里面通过 `h1` 放文章的标题，爬虫将基于这个结构获取文章内容。同样的，为了防止这个，你可以在你的 HTML 添加/删除 额外的标记，周期并且随机的做，eg. 添加额外的 `div` 或 `span`。对于服务端渲染的程序，这应该不会很难。

**注意事项：**

* 它的实现、维护和调试都是很复杂困难的
* 你要注意缓存。特别是你修改你 HTML 元素的 id 或 class 时，也要去修改相应的 CSS 和 JavaScript 文件，这意味着每次修改都要修改这些，而浏览器每次都要重新下载他们。这将导致页面打开慢也会导致服务端负载升高。不过这也不是一个大问题，如果你只是一个星期改变一次。
* 聪明的爬虫仍然能推断出你文章的位置，比如，页面上大块的文本大概率是文章内容。这个让爬虫从页面找到、获取想要的数据。 [Boilerpipe](https://code.google.com/archive/p/boilerpipe/) 就是这样做的。

本质上来说，就是确保爬虫从相似页面获取想要的内容变得不那么容易。

可以参考这个 PHP 的实现：[How to prevent crawlers depending on XPath from getting page contents](http://stackoverflow.com/questions/30361740/)

### 基于用户地理位置修改 HTML

这个和前一个类似。如果你根据不同用户的位置/国家（根据 IP 获取）来提供不同的 HTML，这个可能会破坏将站点 HTML 给用户的爬虫。比如，如果有人写了一个移动 APP 来抓取你的站点，一开始可以用，但是对于不同地区的用户就不起作用了，因为他们会获取到不同的 HTML 结构，嵌入式的 HTML 将不不能正常使用。

### 经常改变 HTML，并与爬虫斗智斗勇！

举个例子：你有一个包含搜索功能的网站，`example.com/search?query=somesearchquery` 的搜索结果是下面的 HTML 结构： 

````html
<div class="search-result">
  <h3 class="search-result-title">Stack Overflow has become the world's most popular programming Q & A website</h3>
  <p class="search-result-excerpt">The website Stack Overflow has now become the most popular programming Q & A website, with 10 million questions and many users, which...</p>
  <a class"search-result-link" href="/stories/stack-overflow-has-become-the-most-popular">Read more</a>
</div>
(And so on, lots more identically structured divs with search results)

````

你可以猜到这个非常容易爬取：一个爬虫需要做的就只是访问搜索链接，然后从返回的 HTML 分析想要的数据。除了定期修改上面 HTML 的内容，你也可以**保留旧结构的 id 和 class，然后使用 CSS 进行隐藏，并使用假数据进行填充，从而给爬虫投毒**。比如像下面这样：

````html
<div class="the-real-search-result">
  <h3 class="the-real-search-result-title">Stack Overflow has become the world's most popular programming Q & A website</h3>
  <p class="the-real-search-result-excerpt">The website Stack Overflow has now become the most popular programming Q & A website, with 10 million questions and many users, which...</p>
  <a class"the-real-search-result-link" href="/stories/stack-overflow-has-become-the-most-popular">Read more</a>
</div>

<div class="search-result" style="display:none">
  <h3 class="search-result-title">Visit example.com now, for all the latest Stack Overflow related news !</h3>
  <p class="search-result-excerpt">EXAMPLE.COM IS SO AWESOME, VISIT NOW! (Real users of your site will never see this, only the scrapers will.)</p>
  <a class"search-result-link" href="http://example.com/">Visit Now !</a>
</div>
(More real search results follow)

````

这意味着基于 id 或 class 获取特定数据的爬虫仍然能继续工作，但是他们将会获取假数据甚至广告，而这些数据真实用户是看不到的，因为他们被 CSS 隐藏了。


### 与爬虫斗智斗勇：在页面插入假的、不可见的蜜罐数据

对上个例子进行补充，你可以在你的 HTML 里面增加不可见的蜜罐数据来抓住爬虫。下面的例子来补充之前说的搜索结果：

``````````
<div class="search-result" style="display:none">
  <h3 class="search-result-title">This search result is here to prevent scraping</h3>
  <p class="search-result-excerpt">If you're a human and see this, please ignore it. If you're a scraper, please click the link below :-)
  Note that clicking the link below will block access to this site for 24 hours.</p>
  <a class"search-result-link" href="/scrapertrap/scrapertrap.php">I'm a scraper !</a>
</div>
(The actual, real, search results follow.)

``````````

一个来获取所有内容的爬虫将会被找到，就像获取其他结果一样，访问链接，查找想要的内容。一个真人将不会看到（因为使用 CSS 隐藏），而且更不会访问这个链接。而正规或者期望的蜘蛛比如谷歌的蜘蛛将不会访问这个链接，因为你可以将 `/scrapertrap/` 加入你的 robots.txt 中（不要忘记增加）

你可以让你的 `scrapertrap.php` 做一些比如限制这个 IP 访问的事情，或者强制这个 IP 之后的请求出验证码。

* 不要忘记在你的 robots.txt 中添加禁止访问 `/scrapertrap/` 的规则避免所有的搜索引擎不会中招
* 你可以/应该和上个经常更换 HTML 结构的例子一起使用
* 也要经常修改它，因为爬虫可能会周期性的了解并不去访问它。修改蜜罐 URL 和文本。同时要考虑使用 id 属性或外部的 CSS 来取代内联的 CSS，要不然爬虫将会学会防止爬取所有包含隐藏属性 `style` 的节点。并且只是偶尔启用它， 这样爬虫一开始正常工作，但是过段时间就不起作用。这个同样也适用于上个例子。
* 要注意可能会有恶意的人在论坛发布像 `[img]http://yoursite.com/scrapertrap/scrapertrap.php[img]` ，然后当正常用户访问轮然然后点击到你的蜜罐链接。所以上个注意事项中经常更换你的蜜罐链接是非常重要的，当然你也可以检查 Referer。

### 当识别为爬虫时，提供假的或无用的数据

如果你确定某个访问是爬虫，你可以提供假的或者无用的数据；这将破坏爬虫从你网站获取的数据。你还应该把它和真实数据进行混淆，这样爬虫就不知道他们获取的到底是真的还是假的数据。

举个例子：如果你有一个新闻站，你检测到了一个爬虫，不要去直接封禁它，而是提供假的或者[随机生成](https://en.wikipedia.org/wiki/Markov_chain#Markov_text_generators)的文章，那爬虫获取的数据将会被破坏。如果你将假数据和真实的数据进行混淆，那爬虫很难获取到他们想要的真实文章。

### 不要接受没有 UserAgent 的请求

很多懒惰的程序员不会在他们的爬虫发请求时带上 UserAgent，而所有的浏览器包括搜索引擎蜘蛛都会携带。

如果请求你时没有携带 UserAgent header 头，你可以展示验证码，或者直接封禁或者限制访问（或者像上面说的提供假数据或者其他的）

这个非常简单去避免，但是作为一种针对书写不当的爬虫，是值得去做的。

### 不要接受 UserAgent 是通用爬虫或在爬虫黑名单的请求

很多情况下，爬虫将会使用真实浏览器或搜索引擎爬虫绝对不会使用的 UserAgent，比如：

* "Mozilla" （就只有这个，我曾经看到过一些爬虫的问题用这个，但真实浏览器绝对不会用）
* "Java 1.7.43_u43" （Java 的 HttpUrlConnection 的默认 UserAgent）
* "BIZCO EasyScraping Studio 2.0"
* "wget", "curl", "libcurl",..  （基础爬虫有时候会用 Wget 和 cURL）

如果你发现一些爬虫使用真实浏览器或合法蜘蛛绝对不会使用的 UserAgent，你可以将其添加到你的黑名单中。


### 检查 Referer header 头

对上一章节的补充，你也可以检查 [Referer] (https://en.wikipedia.org/wiki/HTTP_referer header) （是的，它是 Referer，而不是 Referrer），一些懒惰的爬虫可能不会携带的这个，或者只是每次都携带一样的（有时候是 “google.com”）。举个例子，如果用户是从站内搜索结果页点击进入文章详情的，那要检查 Referer 这个 header 头是否存在还要看搜索结果页的打点。

注意：

* 真实浏览器并不总是携带 Referer；
* 很容易被避免

还是，作为一种防止简单爬虫的方法，也值得去实现。

### 如果不请求资源（CSS，images），它可能不是真实浏览器

一个真实浏览器（通常）会请求和下载资源比如 CSS 和 图片。HTML 解析器和爬取器可能只关心特定的页面和内容。

你可以基于访问你资源的日志，如果你看到很多请求只请求 HTML，那它可能就是爬虫。

注意搜索引擎蜘蛛、旧的移动设备、屏幕阅读器和设置错误的设备可能也不会请求资源。


### 要求使用 Cookie；使用它们来跟踪用户和爬虫行为

访问你网站时，你可以要求 cookie 必须开启。这个能识别没有经验和爬虫新手，然而爬虫要携带 Cookie 也十分简单。如果你要求开启 Cookie，你能使用它们来追踪用户和爬虫的行为，然后基于此来实现频率限制、封禁、或者显示验证码而不仅仅依赖 IP 地址。

举个例子： 当用户进行搜索时，设置一个唯一的 Cookie。当搜索结果加载出来之后，验证这个 Cookie。如果一个用户打开了所有的搜索结果（可以从 Cookie 中得知），那很可能就是爬虫

使用 Cookie 可能是低效的，因为爬虫也可以携带 Cookie 发送请求，也可以根据需要丢弃。如果你的网站只能在开启 Cookie 时使用，你也不能给关闭 Cookie 的用户提供服务。

要注意如果你使用 JavaScript 去设置和检测 Cookie，你能封禁那些没有运行 JavaScript 的爬虫，因为它们没办法获取和发送 Cookie。

### 使用 JavaScript 和 AJAX 加载内容

你可以在页面加载完成之后，使用 JavaScript + AJAX 来加载你的内容。这个对于那些没有运行 JavaScript 的 HTML 分析器来说将无法取得数据。这个对于没有经验或者新手程序员写的爬虫非常有效。

注意：

* 使用 JavaScript 加载内容将会降低用户体验和性能；
* 搜索引擎也不会运行 JavaScript，因此不会对你的内容进行收录。这对于搜索结果来说可能不是问题，但是要注意其他页面，比如文章页面；
* 写爬虫的程序员获取到加载内容的 API 后可以直接使用它

### 混淆你的数据和网络请求，不要让其直接通过脚本就能获取

如果你用 Ajax 和 JavaScript 加载你的数据，在传输的时候要混淆一下。比如，你可以在服务器端 encode 你的数据（比如简单的使用 base64 或 负载一些的多次混淆、位偏移或者是进行加密），然后在客户端在 Ajax 获取数据之后再进行 decode。这意味着如果有人要抓包获取你的请求就不能直接看到你页面如何加载数据，而且那些人也不能直接通过 API 获得你的数据，如果想要获取数据，就必须要去解密你的算法。

* 如果你用 Ajax 加载数据，那你应该强制在页面加载之后才可以获取，比如要求获取数据必须包含 session 信息，这些你可以在页面加载的时候嵌入到 JavaScript 或 HTML 中

* 你也可以直接吧混淆的数据嵌入到 HTML 中，然后用 JavaScript 去解密然后再显示它们，这样的话，你就不需要再使用 Ajax 去做额外的请求。这样做可以让那些不运行 JavaScript 的 HTML 解析器更难获取你的数据，他们必须要反解你的 JavaScript（没错，JavaScript 也要做混淆）

* 你应该经常更换混淆方法以免爬虫找出方法揭秘它

下面是一些这个方式的缺点：

* 实现、维护和调试都非常麻烦
* **虽然让爬虫变得不容易抓取，但是对于截屏类的爬虫来说，它们实际上会运行 JavaScript，所以能获取到数据**（不过很多简单的 HTML 解释器不会运行 JavaScript）
* 如果真实用户禁用了 JavaScript，那你的网站将不能正常显示
* 性能和页面加载速度会受到影响

## 其他非技术做法

### 你的服务器供应商可能提供搜索引擎蜘蛛或爬虫的防御：

比如，CloudFlare 提供反蜘蛛和反爬虫的防御，你只需要直接启用它就可以了，另外 AWS 也提供类似服务。而且 Apache 的 mod_evasive 模块也能让你很轻松地实现频率限制。

### 直接告诉别人不要抓取，会有人尊重而且停止抓取

你应该直接告诉人们不要抓取你的网站，比如，在你的服务条款中表明。有些人确实会尊重它，而且不在你允许的情况下不会再去抓取数据。

### 寻求律师的援助

律师们知道如何处理侵犯版权的事情，而且他们可以发送律师函。DMCA（译者注：Digital Millennium Copyright Act，数字千年版权法案，是一个美国版权法律） 也能提供援助。


### 直接提供 API 获取你的数据

这看起来适得其反，但是你可以要求标明来源并包含返回你站点的链接。甚至也可以售卖你的 API 而赚取费用。

还有就是，Stack Exchange 提供了 API，但是必须要标明来源。

## 其他补充

* 要在用户体验和反扒之间做权衡：你做的每个举措都有可能在某种程度上影响用户体验，所以你必须要权衡和妥协；

* 不要忘你的移动站点和 APP：如果你的站点有移动版，要小心爬虫也可以通过它爬取你的数据。或者说你有移动 APP，他们也可以截屏分析，或者可以抓取你的网络请求去直接找到你的 RESTful 的 API 直接使用；

* 如果你对一些特定浏览器提供了特定的版本，比如对很老的 IE 版本提供网站的阉割版，不要忘记爬虫也可以直接爬取它；

* 选出集中最适合你的策略结合起来使用，而不是只用一种；

* 爬虫可以抓取其他爬虫：如果有个网站显示的内容都是从你网站爬取的，那另外一个爬虫也可以直接爬取那个网站。

## 有哪些最有效的方法 ？

以我自己写和帮忙别人写爬虫的经验，我认为最有效的方法是：

* 经常修改 HTML 的结构

* 蜜罐和假数据

* 使用混淆的 JavaScript、Ajax 还有 Cookie

* 频率限制、爬虫检测和请求封禁

扩展阅读：
 
* [维基百科关于 Web 爬虫的文章](http://en.wikipedia.org/wiki/Web_scraping)，其中提到了很多 Web 爬虫的相关技术和爬虫类型，看完如何进行 web 爬取的一些信息，也不要忘记看一看爬取的合法性。

**最后祝你在保护你网站的内容坎坷路上一路顺风...**





















