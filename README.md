#A guide to preventing Webscraping

**(Or at least making it harder)**

---

Note: this is an expanded version of my answer on Stack Overflow [here](http://stackoverflow.com/a/34828465/4428462), I've put it here on GitHub since it's too long for SO (30k characters is the max, this is over 40k chars).

---

**Essentially, hindering scraping means that you need to make it difficult for scripts and machines to get the wanted data from your website, while not making it difficult for real users and search engines**.

 Unfortunately this is hard, and you will need to make trade-offs between preventing scraping and degrading the accessibility for real users and search engines.

In order to hinder scraping (also known as _Webscraping_ _Screenscraping_, _web data mining_, _web harvesting_, or _web data extraction_), it helps to know how these scrapers work, and what prevents them from working well, and this is what this answer is about.

Generally, these scraper programs are written in order to extract specific information from your site, such as articles, search results, product details, or in your case, artist and album information. Usually, people scrape websites for specific data, in order to reuse it on their own site (and make money out of your content !), or to build alternative frontends for your site (such as mobile apps), or even just for private research or analysis purposes.

Essentially, there are various types of scraper, and each works differently:

* Spiders, such as [Google's bot](http://googlebot.com) or website copiers like [HTtrack](http://www.httrack.com), which visit your website, and recursively follow links to other pages in order to get data. These are sometimes used for targeted scraping to get specific data, often in combination with a HTML parser to extract the desired data from each page.

* Shell scripts: Sometimes, common Unix tools are used for scraping: Wget or Curl to download pages, and Grep (Regex) to extract the desired data, usually using a shell script. These are the simplest kind of scraper, and also the most fragile kind ([Don't ever try parse HTML with regex !](http://example.com)). These are thus the easiest kind of scraper to break and screw with.

* HTML scrapers and parsers, such as ones based on [Jsoup](http://jsoup.org), [Scrapy](http://scrapy.org/), and many others. Similar to shell-script regex based ones, these work by extracting data from your pages based on patterns in your HTML, usually ignoring everything else. 

    So, for example: If your website has a search feature, such a scraper might submit a HTTP request for a search, and then get all the result links and their titles from the results page HTML, sometimes hundreds of times for hundreds of different searches, in order to specifically get only search result links and their titles. These are the most common.

* Screenscrapers, based on eg. [Selenium](http://www.seleniumhq.org/) or [PhantomJS](http://phantomjs.org), which actually open your website in a real browser, run JavaScript, AJAX, and so on, and then get the desired text from the webpage, usually by:

    * Getting the HTML from the browser after your page has been loaded and JavaScript has run, and then using a HTML parser to extract the desired data or text. These are the most common, and so many of the methods for breaking HTML parsers / scrapers also work here.

    * Taking a screenshot of the rendered pages, and then using OCR to extract the desired text from the screenshot. These are rare, and only dedicated scrapers who really want your data will set this up.

    Browser-based screenscrapers harder to deal with, as they run scripts, render HTML, and can behave like a real human browsing your site.

* Webscraping services such as [ScrapingHub](http://scrapinghub.com/) or [Kimono](https://www.kimonolabs.com/). In fact, there's people whose job is to figure out how to scrape your site and pull out the content for others to use. These sometimes use large networks of proxies and ever changing IP addresses to get around limits and blocks, so they are especially problematic.

    Unsurprisingly, professional scraping services are the hardest to deter, but if you make it hard and time-consuming to figure out how to scrape your site, these (and people who pay them to do so) may not be bothered to scrape your website.

* Embedding your website in other site's pages with [frames](https://en.wikipedia.org/wiki/Framing_(World_Wide_Web)), and embedding your site in mobile apps. 

     While not technically scraping, this is also a problem, as mobile apps (Android and iOS) can embed your website, and even inject custom CSS and JavaScript, thus completely changing the appearance of your site, and only showing the desired information, such as the article content itself or the list of search results, and hiding things like headers, footers, or ads.

* Human copy - and - paste: People will copy and paste your content in order to use it elsewhere. Unfortunately, there's not much you can do about this.

There is a lot overlap between these different kinds of scraper, and many scrapers will behave similarly, even though they use different technologies and methods to get your content.

This collection of tips are mostly my own ideas, various difficulties that I've encountered while writing scrapers, as well as bits of information and ideas from around the interwebs. Please do edit and add your own.

##So, how do you prevent scraping ?

##General:

Some general methods to detect and deter scrapers:

###Monitor your logs & traffic patterns; limit access if you see unusual activity:

It is a good idea to check your logs regularly, and in case of unusual activity indicative of automated access (scrapers), such as many similar actions from the same IP address, you can block or limit access.

Specifically, some ideas:

* **Rate limiting:**

    Only allow users (and scrapers) to perform a limited number of actions in a certain time - for example, only allow a few searches per second from any specific IP address or user. This will slow down scrapers, and make them less effective. You could also show a captcha if actions are completed too fast or faster than a real user would.

* **Detect unusual activity:**

    If you see unusual activity, such as many similar requests from a specific IP address, someone looking at an excessive number of pages or performing an unusual number of searches, you can prevent access, or show a captcha for subsequent requests.

* **Don't just monitor & rate limit by IP address - use other indicators too:**

    If you do block or rate limit, don't just do it on a per-IP address basis; you can use other indicators and methods to identify specific users or scrapers. Some indicators which can help you identify specific users / scrapers include:

    * How fast users fill out forms, and where on a button they click;

    * You can gather a lot of information with JavaScript, such as screen size / resolution, timezone, installed fonts, etc; you can use this to identify users.

    * Http headers and their order, especially User-Agent.

    As an example, if you get many request from a single IP address, all using the same User agent, screen size (determined with JavaScript), and the user (scraper in this case) always clicks on the button in the same way and at regular intervals, it's probably a screen scraper; and you can temporarily block similar requests (eg. block all requests with that user agent and screen size coming from that particular IP address), and this way you won't inconvenience real users on that IP address, eg. in case of a shared internet connection.

    You can also take this further, as you can identify similar requests, even if they come from different IP addresses, indicative of distributed scraping (a scraper using a botnet or a network of proxies). If you get a lot of otherwise identical requests, but they come from different IP addresses, you can block. Again, be aware of not inadvertently blocking real users.

    This can be effective against screenscrapers which run JavaScript, as you can get a lot of information from them.

    Related questions on Security Stack Exchange:

    * [How to uniquely identify users with the same external IP address?](http://security.stackexchange.com/questions/81302/how-to-uniquely-identify-users-with-the-same-external-ip-address) for more details, and 

    * [Why do people use IP address bans when IP addresses often change?](http://security.stackexchange.com/questions/96377/why-do-people-use-ip-address-bans-when-ip-addresses-often-change) for info on the limits of these methods.

* **Instead of temporarily blocking access, use a Captcha:**

    The simple way to implement rate-limiting would be to temporarily block access for a certain amount of time. However, this has problems: if for some reason you do temporarily block a user, even temporarily (or multiple users, since IP addresses or devices can be shared), they will be irritated, and may leave.

    Instead, do what Google does: after users go over your limits, _force a Captcha for subsequent requests_. If the user solves the captcha, they're (probably) not a scraper, and you can then allow access again (possibly with more stringent anti-scraping measures, such as showing another captcha sooner if the user keeps going). If they're a scraper and can't solve the captcha, well, you've succeeded at keeping the scraper out.

   Beware that Captchas are not infallible, see the section on Captchas further down.

###Require registration & login

A possibility would be to require account creation in order to view your content, if this is feasible for your site. This is a good deterrent for scrapers, but is unfortunately also a good deterrent for real users.

* If you require account creation and login, you can accurately track user and scraper actions. That way, you can easily detect when a specific account is being used for scraping, and delete it. Things like rate limiting or detecting abuse (such as a huge number of searches in a short time by a specific user) become much easier, as you can identify specific scrapers instead of just IP addresses.

In order to avoid scripts creating many accounts, you should:

* Require an email address for registration, and verify that email address by sending a link that must be opened in order to activate the account. Allow only one account per email address.

* Require a captcha to be solved during registration / account creation, again to prevent scripts from creating accounts.

Unfortunately, requiring account creation to view content will drive users away. For example, if you require account creation in order to view an article, users will just go elsewhere.

###Block access from cloud hosting and scraping service IP addresses

Sometimes, scrapers will be run from web hosting services, such as Amazon Web Services or Google app Engine, or Virtual Private Servers. You can block / limit access to your website (or show a captcha) for requests originating from the IP addresses used by such cloud hosting services. You can also block access from IP addresses used by scraping services (such as the aforementioned Kimono).

Similarly, you can also limit access from IP addresses used by proxy or VPN providers, as scrapers may use such proxy servers to avoid many requests being detected.

Beware that by blocking access from proxy servers and VPNs, you will negatively affect real users.

###Make your error message nondescript if you do block

If you do block / limit access, you should ensure that you don't tell the scraper what caused the block, thereby giving them clues as to how to fix their scraper. So a bad idea would be to show error pages with text like:

* Error, too many requests from your IP address, please try again later.

* Error, User Agent header not present, request blocked !

Instead, show a friendly error message that doesn't tell the scraper what caused it. Something like this is much better:

* Sorry, something went wrong. You can contact support via `helpdesk@example.com`, should the problem persist. Please mention your unique support key `9ac3f01`, so we can diagnose & resolve the problem.

This is also a lot more user friendly for real users, should they ever see such an error page. You should also consider showing a captcha for subsequent requests instead of a hard block, in case a real user sees the error message, so that you don't block and thus cause legitimate users to contact you.

###Use Captchas if you suspect that your website is being accessed by a scraper.

Captchas ("Completely Automated Test to Tell Computers and Humans apart") are very effective against stopping scrapers. Unfortunately, they are also very effective at irritating users. 

As such, they are useful when you suspect a possible scraper, and want to stop the scraping, without also blocking access in case it isn't a scraper but a real user. You might want to consider showing a captcha before allowing access to the content if:

* You have many requests coming from the same IP address, more than a real user could generate (remember that IP addresses are often shared between multiple users, as also described). Google does this, for example.

* You get requests or a series of requests which a real user would be unlikely to perform, such as performing a search (or multiple searches in a row) and then visiting all the result links.

* Assets such as CSS are not requested.

* Etc..

Things to be aware of when using Captchas:

* Don't roll your own, use something like Google's [reCaptcha](https://www.google.com/recaptcha/intro/index.html) : It's a lot easier than implementing a captcha yourself, it's more user-friendly than some blurry and warped text solution you might come up with yourself (users often only need to tick a box), and it's also a lot harder for a scripter to solve than a simple image served from your site

* Don't include the solution to the captcha in the HTML markup: I've actually seen one website which had the solution for the captcha _in the page itself_, (although quite well hidden) thus making it pretty useless. Don't do something like this. Again, use a service like reCaptcha, and you won't have this kind of problem (if you use it properly).

* Captchas can be solved in bulk: There are captcha-solving services where actual, low-paid, humans solve captchas in bulk. Again, using reCaptcha is a good idea here, as they have protections (such as the relatively short time the user has in order to solve the captcha). This kind of service is unlikely to by used unless your data is really valuable.

###Serve your text content as an image

You can render text into an image server-side, and serve that to be displayed, which will hinder simple scrapers extracting text.

 However, this is bad for screen readers, search engines, performance, and pretty much everything else. It's also illegal in some places (due to accessibility, eg. the Americans with Disabilities Act), and it's also easy to circumvent with some OCR, so don't do it. 

You can do something similar with CSS sprites, but that suffers from the same problems.

###Don't expose your complete dataset:

If feasible, don't provide a way for a script / bot to get all of your dataset. As an example: You have a news site, with lots of individual articles. You could make those articles be only accessible by searching for them via the on site search, and, if you don't have a list of _all_ the articles on the site and their URLs anywhere, those articles will be only accessible by using the search feature. This means that a script wanting to get all the articles off your site will have to do searches for all possible phrases which may appear in your articles in order to find them all, which will be time-consuming, horribly inefficient, and will hopefully make the scraper give up.

This will be ineffective if:

* The bot / script does not want / need the full dataset anyway.
* Your articles are served from a URL which looks something like `example.com/article.php?articleId=12345`. This (and similar things) which will allow scrapers to simply iterate over all the `articleId`s and request all the articles that way.
* There are other ways to eventually find all the articles, such as by writing a script to follow links within articles which lead to other articles.
* Searching for something like "and" or "the" can reveal almost everything, so that is something to be aware of. (You could / should avoid this by only returning the top 10 or 20 results).
* You need search engines to find your content.

An example of a site which does this is Google News.

###Don't expose your APIs, endpoints, and similar things:

Make sure you don't expose any APIs, even unintentionally. For example, if you are using AJAX or network requests from within Adobe Flash or Java Applets (God forbid!) to load your data it is trivial to look at the network requests from the page and figure out where those requests are going to, and then reverse engineer and use those endpoints in a scraper program. Make sure you obfuscate your endpoints and make them hard for others to use, as also described below.

##To deter HTML parsers and scrapers:

Since HTML parsers work by extracting content from pages based on identifiable patterns in the HTML, we can intentionally change those patterns in oder to break these scrapers, or even screw with them. Most of these tips also apply to other scrapers like spiders and screenscrapers too. Some ideas:

###Frequently change your HTML

Scrapers which process HTML directly do so by extracting contents from specific, identifiable parts of your HTML page. For example: If all pages on your website have a `div` with an id of `article-content`, which contains the text of the article, then it is trivial to write a script to visit all the article pages on your site, and extract the content text of the `article-content ` div on each article page, and voilà, the scraper has all the articles from your site in a format that can be reused elsewhere.

If you change the HTML and the structure of your pages frequently, such scrapers will no longer work.

* You can frequently change the id's and classes of elements in your HTML, perhaps even automatically. So, if your `div.article-content` becomes something like `div.a4c36dda13eaf0`, and changes every week, the scraper will work fine initially, but will break after a week. Make sure to change the length of your ids / classes too, otherwise the scraper will use `div.[any-14-characters]` to find the desired div instead. Beware of other similar holes too..

* If there is no way to find the desired content from the markup, the scraper will do so from the way the HTML is structured. So, if all your article pages are similar in that every `div` inside a `div` which comes after a `h1` is the article content, scrapers will get the article content based on that. Again, to break this, you can add / remove extra markup to your HTML, periodically and randomly, eg. adding extra `div`s or `span`s. With modern server side HTML processing, this should not be too hard.

Things to be aware of:

* It will be tedious and difficult to implement, maintain, and debug.

* You will hinder caching. Especially if you change ids or classes of your HTML elements, this will require corresponding changes in your CSS and JavaScript files, which means that every time you change them, they will have to be re-downloaded by the browser. This will result in longer page load times for repeat visitors, as well as increased server load. Although if you only change it once a week, it will not be a big problem.

* Clever scrapers will still be able to get your content by inferring where the actual content is, eg. by knowing that a large single block of text on the page is likely to be the actual article, or that a list of many similarly structured items is likely to be a search results page, and each item in the list is likely to contain the desired data. This makes it possible to still find & extract the desired data from the page. [Boilerpipe](https://code.google.com/archive/p/boilerpipe/) does exactly this.

Essentially, make sure that it is not easy for a script to find the actual, desired content for every similar page.

See also [How to prevent crawlers depending on XPath from getting page contents](http://stackoverflow.com/questions/30361740/) for details on how this can be implemented in PHP.


###Change your HTML based on the user's location

This is sort of similar to the previous tip. If you serve different HTML based on your user's location / country (determined by IP address), this may break scrapers which are delivered to users. For example, if someone is writing a mobile app which scrapes data from your site, it will work fine initially, but break when it's actually distributed to users, as those users may be in a different country, and thus get different HTML, which the embedded scraper was not designed to consume.

###Frequently change your HTML, actively screw with the scrapers by doing so !

An example: You have a search feature on your website, located at `example.com/search?query=somesearchquery`, which returns the following HTML:

    <div class="search-result">
      <h3 class="search-result-title">Stack Overflow has become the world's most popular programming Q & A website</h3>
      <p class="search-result-excerpt">The website Stack Overflow has now become the most popular programming Q & A website, with 10 million questions and many users, which...</p>
      <a class"search-result-link" href="/stories/stack-overflow-has-become-the-most-popular">Read more</a>
    </div>
    (And so on, lots more identically structured divs with search results)

As you may have guessed this is easy to scrape: all a scraper needs to do is hit the search URL with a query, and extract the desired data from the returned HTML. In addition to periodically changing the HTML as described above, you could also **leave the old markup with the old ids and classes in, hide it with CSS, and fill it with fake data, thereby poisoning the scraper.** Here's how the search results page could be changed:

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

This will mean that scrapers written to extract data from the HTML based on classes or IDs will continue to seemingly work, but they will get fake data or even ads, data which real users will never see, as they're hidden with CSS.

###Screw with the scraper: Insert fake, invisible honeypot data into your page

Adding on to the previous example, you can add invisible honeypot items to your HTML to catch scrapers. An example which could be added to the previously described search results page:

    <div class="search-result" style=”display:none">
      <h3 class="search-result-title">This search result is here to prevent scraping</h3>
      <p class="search-result-excerpt">If you're a human and see this, please ignore it. If you're a scraper, please click the link below :-)
      Note that clicking the link below will block access to this site for 24 hours.</p>
      <a class"search-result-link" href="/scrapertrap/scrapertrap.php">I'm a scraper !</a>
    </div>
    (The actual, real, search results follow.)

A scraper written to get all the search results will pick this up, just like any of the other, real search results on the page ..and visit the link, looking for the desired content. A real human  will never even see it in the first place (due to it being hidden with CSS), and won't visit the link. A genuine and desirable spider such as Google's will not visit the link either because you disallowed `/scrapertrap/` in your robots.txt (don't forget this!)

You can make your `scrapertrap.php` do something like block access for the IP address that visited it, force a captcha for all subsequent requests from that IP, create a loop of infinite redirects, or whatever.

* Don't forget to disallow your honeypot (`/scrapertrap/`) in your robots.txt file so that search engine bots don't fall into it.

* You can / should combine this with the previous tip of changing your HTML frequently.

* Change this frequently too, as scrapers will eventually learn to avoid it. Change the honeypot URL, and the text. You might also want to consider changing the inline CSS used for hiding, and use an ID attribute and external CSS instead, as scrapers will learn to avoid anything which has a `style` attribute with CSS used to hide the content. You might also want to try only enabling it sometimes, so the scraper works initially, but breaks after a while. This also applies to the previous tip.

###Serve fake and useless data if you detect a scraper

Again adding on to the previous items, if you detect what is obviously a scraper, you can serve up fake and useless data; this will corrupt the data the scraper gets from your website. You should also make it impossible to distinguish such fake data from real data, so that scrapers don't know that they're being screwed with.

As an example: if you have a news website; if you detect a scraper, instead of blocking access, just serve up fake, [randomly generated](https://en.wikipedia.org/wiki/Markov_chain#Markov_text_generators) articles, and this will poison the data the scraper gets. If you make your faked data or articles indistinguishable from the real thing, you'll make it hard for scrapers to get what they want, namely the actual, real articles. 

###Don't accept requests if the User Agent is empty / missing

Often, lazily written scrapers will not send a User Agent header with their request, whereas all  browsers as well as search engine spiders will. 

If you get a request where the User Agent header is not present, you can show a captcha, or simply block or limit access. (Or serve fake data as described above, or something else..)

Of course, this would is trivial to circumvent by sending a valid user agent string used by browsers, but as an additional measure against poorly written scrapers it is worth implementing.

###Don't accept requests if the User Agent is a common scraper one; blacklist ones used by scrapers

In some cases, scrapers will use a User Agent which no real browser or search engine spider uses, such as:

*  "Mozilla" (Just that, nothing else. I've seen a few questions about scraping here, using that. A real browser will never use only that single word.)
*  "Java 1.7.43_u43" (By default, Java's HttpUrlConnection uses something like this.)
*  "BIZCO EasyScraping Studio 2.0"
*  "wget", "curl", "libcurl",.. (Wget and cURL are sometimes used for basic scraping)

If you find that a specific User Agent string is used by scrapers on your site, and it is not used by real browsers or legitimate spiders, you can also add it to your blacklist.

Again, you could block / limit / show a captcha for requests with such User Agents. Again, this is easy to circumvent, and if there are real users using such a user agent, they will be affected too,

###Check the Referer header

Adding on to the previous item, you can also check for the [Referer](https://en.wikipedia.org/wiki/HTTP_referer header) (yes, it's Referer, not Referrer), as lazily written scrapers may not send it, or always send the same thing (sometimes "google.com"). As an example, if the user comes to an article page from a on-site search results page, check that the Referer header is present and points to that search results page.

Beware that:

* Real browsers don't always send it either;

* It's trivial to spoof.

Again, as an additional measure against poorly written scrapers it may be worth implementing.

###If it doesn't request assets (CSS, images), it's not a real browser.

A real browser will (almost always) request and download assets such as images and CSS. HTML parsers and scrapers won't as they are only interested in the actual pages and their content.

You could log requests to your assets, and if you see lots of requests for only the HTML, block, slow down access, or show a captcha.

Beware that:

* Search engine bots and legitimate spiders may not request your CSS, images, or other assets either.

* Screen readers, text-only browsers, ancient mobile browsers, and misconfigured devices may not request your assets.

* If the user has turned off images or CSS styling, their browser will not request these.

###Use and require cookies; use them to track user and scraper actions.

You can require cookies to be enabled in order to view your website. This will deter inexperienced and newbie scraper writers, however it is easy to for a scraper to send cookies. If you do use and require them, you can track user and scraper actions with them, and thus implement rate-limiting, blocking, or showing captchas on a per-user instead of a per-IP basis.

For example: when the user performs search, set a unique identifying cookie. When the results pages are viewed, verify that cookie. If the user opens all the search results (you can tell from the cookie), then it's probably a scraper.

Using cookies may be ineffective, as scrapers can send the cookies with their requests too, and discard them as needed. You will also prevent access for real users who have cookies disabled, if your site only works with cookies enabled.

Note that if you use JavaScript to set and retrieve the cookie, you'll block scrapers which don't run JavaScript, since they can't retrieve and send the cookie with their request.

###Use JavaScript + Ajax to load your content

You could use JavaScript + AJAX to load your content after the page itself loads. This will make the content inaccessible to HTML parsers which do not run JavaScript. This is often an effective deterrent to newbie and inexperienced programmers writing scrapers.

Things to be aware of:

* Using JavaScript to load the actual content will degrade user experience and performance (Who wants to wait for a search results page to load and then wait again for the content to be fetched and loaded ?).

* Search engines may not run JavaScript either (I'm not quite sure about this), thus preventing them from indexing your content. This may not be a problem for search results pages, but may be for other things, such as article pages.

* A programmer writing a scraper who knows what they're doing can discover the endpoints where the content is loaded from and use them. See also the next section on obfuscating your endpoints and the data available from them.

###Obfuscate your markup, network requests from scripts, and everything else.

If you use Ajax and JavaScript to load your data, you can obfuscate the data which is transferred. As an example, you could encode your data on the server (with something as simple as base64 or more complex with multiple layers of obfuscation, bit-shifting, and maybe even encryption), and then decode and display it on the client, after fetching via Ajax. This will mean that someone inspecting network traffic will not immediately see how your page works and loads data, and it will be tougher for someone to directly request request data from your endpoints, as they will have to reverse-engineer your descrambling algorithm.

* If you do use Ajax for loading the data, you should make it hard to use the endpoints without loading the page first, eg by requiring some session key as a parameter, which you can embed in your JavaScript or your HTML.

* You can also embed your obfuscated data directly in the initial HTML page and use JavaScript to deobfuscate and display it, which would avoid the extra network requests. Doing this will make it significantly harder to extract the data using a HTML-only parser which does not run JavaScript, as the one writing the scraper will have to reverse engineer your JavaScript (which you should obfuscate too).

* You might want to change your obfuscation methods regularly, to break scrapers who have figured it out.

There are several disadvantages to doing something like this, though:

* It will be tedious and difficult to implement, maintain, and debug.

* **It will be ineffective against scrapers and screenscrapers which actually run JavaScript and then extract the data**. (Most simple HTML parsers don't run JavaScript though)

* It will make your site nonfunctional for real users if they have JavaScript disabled.

* Performance and page-load times will suffer.

##Spiders:

Spiders are different from HTML scrapers in that they do not only seek to extract specific information from specific parts of your site, but that they often follow all links on your pages and attempt to find all the pages on your site. A typical use case would be to copy or mirror an entire website.

In addition to the points made above, which are also effective against spiders, it's worth noting that:

* Honeypots (as described above) can be especially effective. Add a link somewhere in your pages, which is invisible to humans (hide it with CSS), and which will not be followed by legitimate spiders (eg. Googles's), as legitimate spiders will respect your robots.txt, while spiders seeking to rip off your site may not. When something visits that link, you can log the IP address and block access or show a captcha for subsequent requests. Beware that sometimes, even undesirable spiders will respect your robots.txt, so this may not be entirely effective. The [top voted answer](http://stackoverflow.com/a/3161695/) explains this very well.

* Since most simple spiders don't run JavaScript, if you require JavaScript in order to load your content, those spiders will not get access to the content. See also my point about JavaScript above.

##Screenscrapers and scraping services:

Unfortunately, since these actually load your site in a real browser, run JavaScript, and so on, and sometimes even take a screenshot of your page and use OCR to get the desired data, these can be difficult or impossible deal with. However, you should still use rate limits, captchas, and restrict access when you get too many requests from a single IP address.

In addition, if the scraper does directly extract data from your HTML, the above techniques for HTML scrapers are also effective. To prevent OCR, you can change your page layout and appearance every once in a while, bit that's both extremely tedious and extremely irritating for users.

##Non-Technical:

###Tell people not to scrape, and some will respect it

You should tell people not to scrape your site, eg. in your conditions or Terms Of Service. Some people will actually respect that, and not scrape data from your website without permission.

###Find a lawyer

They know how to deal with copyright infringement, and can send a cease-and-desist letter. The DMCA is also helpful in this regard.

This is the approach Stack Overflow and Stack Exchange uses.

###Make your data available, provide an API:

This may seem counterproductive, but you could make your data easily available and require attribution and a link back to your site. Maybe even charge $$$ for it..

Again, Stack Exchange provides an API, but with attribution required.

##Miscellaneous:

* Find a balance between usability for real users and scraper-proofness: Everything you do will impact user experience negatively in one way or another, so you will need to find compromises.

* Don't forget your mobile site and apps: If you have a mobile version of your site, beware that scrapers can also scrape that. If you have a mobile app, that can be screen scraped too, and network traffic can be inspected to figure out the REST endpoints it uses.

* If you serve a special version of your site for specific browsers, eg. a cut-down version for older versions of Internet Explorer, don't forget that scrapers can scrape that, too.

* Use these tips in combination, pick what works best for you.

* Scrapers can scrape other scrapers: If there is one website which shows content scraped from your website, other scrapers can scrape from that scraper's website.

##What's the most effective way ?

In my experience of writing scrapers and helping people to write scrapers here on SO, the most effective methods are :

* Changing the HTML markup frequently

* Honeypots and fake data

* Using obfuscated JavaScript, AJAX, and Cookies

* Rate limiting and scraper detection and subsequent blocking.

##Further reading:

* [Wikipedia's article on Web scraping](http://en.wikipedia.org/wiki/Web_scraping). Many details on the technologies involved and the different types of web scraper, general information on how webscraping is done, as well as a look at the legalities of scraping.

**Good luck on the perilous journey of protecting your content...**
example.com`, should the problem persist. Please mention your unique support key `9ac3f01`, so we can diagnose & resolve the problem.

This is also a lot more user friendly for real users, should they ever see such an error page. You should also consider showing a captcha for subsequent requests instead of a hard block, in case a real user sees the error message, so that you don't block and thus cause legitimate users to contact you.

###Use Captchas if you suspect that your website is being accessed by a scraper.

Captchas ("Completely Automated Test to Tell Computers and Humans apart") are very effective against stopping scrapers. Unfortunately, they are also very effective at irritating users. 

As such, they are useful when you suspect a possible scraper, and want to stop the scraping, without also blocking access in case it isn't a scraper but a real user. You might want to consider showing a captcha before allowing access to the content if:

* You have many requests coming from the same IP address, more than a real user could generate (remember that IP addresses are often shared between multiple users, as also described). Google does this, for example.

* You get requests or a series of requests which a real user would be unlikely to perform, such as performing a search (or multiple searches in a row) and then visiting all the result links.

* Assets such as CSS are not requested.

* Etc..

Things to be aware of when using Captchas:

* Don't roll your own, use something like Google's [reCaptcha](https://www.google.com/recaptcha/intro/index.html) : It's a lot easier than implementing a captcha yourself, it's more user-friendly than some blurry and warped text solution you might come up with yourself (users often only need to tick a box), and it's also a lot harder for a scripter to solve than a simple image served from your site

* Don't include the solution to the captcha in the HTML markup: I've actually seen one website which had the solution for the captcha _in the page itself_, (although quite well hidden) thus making it pretty useless. Don't do something like this. Again, use a service like reCaptcha, and you won't have this kind of problem (if you use it properly).

* Captchas can be solved in bulk: There are captcha-solving services where actual, low-paid, humans solve captchas in bulk. Again, using reCaptcha is a good idea here, as they have protections (such as the relatively short time the user has in order to solve the captcha). This kind of service is unlikely to by used unless your data is really valuable.

###Serve your text content as an image

You can render text into an image server-side, and serve that to be displayed, which will hinder simple scrapers extracting text.

 However, this is bad for screen readers, search engines, performance, and pretty much everything else. It's also illegal in some places (due to accessibility, eg. the Americans with Disabilities Act), and it's also easy to circumvent with some OCR, so don't do it. 

You can do something similar with CSS sprites, but that suffers from the same problems.

###Don't expose your complete dataset:

If feasible, don't provide a way for a script / bot to get all of your dataset. As an example: You have a news site, with lots of individual articles. You could make those articles be only accessible by searching for them via the on site search, and, if you don't have a list of _all_ the articles on the site and their URLs anywhere, those articles will be only accessible by using the search feature. This means that a script wanting to get all the articles off your site will have to do searches for all possible phrases which may appear in your articles in order to find them all, which will be time-consuming, horribly inefficient, and will hopefully make the scraper give up.

This will be ineffective if:

* The bot / script does not want / need the full dataset anyway.
* Your articles are served from a URL which looks something like `example.com/article.php?articleId=12345`. This (and similar things) which will allow scrapers to simply iterate over all the `articleId`s and request all the articles that way.
* There are other ways to eventually find all the articles, such as by writing a script to follow links within articles which lead to other articles.
* Searching for something like "and" or "the" can reveal almost everything, so that is something to be aware of. (You could / should avoid this by only returning the top 10 or 20 results).
* You need search engines to find your content.

An example of a site which does this is Google News.

###Don't expose your APIs, endpoints, and similar things:

Make sure you don't expose any APIs, even unintentionally. For example, if you are using AJAX or network requests from within Adobe Flash or Java Applets (God forbid!) to load your data it is trivial to look at the network requests from the page and figure out where those requests are going to, and then reverse engineer and use those endpoints in a scraper program. Make sure you obfuscate your endpoints and make them hard for others to use, as also described below.

##To deter HTML parsers and scrapers:

Since HTML parsers work by extracting content from pages based on identifiable patterns in the HTML, we can intentionally change those patterns in oder to break these scrapers, or even screw with them. Most of these tips also apply to other scrapers like spiders and screenscrapers too. Some ideas:

###Frequently change your HTML

Scrapers which process HTML directly do so by extracting contents from specific, identifiable parts of your HTML page. For example: If all pages on your website have a `div` with an id of `article-content`, which contains the text of the article, then it is trivial to write a script to visit all the article pages on your site, and extract the content text of the `article-content ` div on each article page, and voilà, the scraper has all the articles from your site in a format that can be reused elsewhere.

If you change the HTML and the structure of your pages frequently, such scrapers will no longer work.

* You can frequently change the id's and classes of elements in your HTML, perhaps even automatically. So, if your `div.article-content` becomes something like `div.a4c36dda13eaf0`, and changes every week, the scraper will work fine initially, but will break after a week. Make sure to change the length of your ids / classes too, otherwise the scraper will use `div.[any-14-characters]` to find the desired div instead. Beware of other similar holes too..

* If there is no way to find the desired content from the markup, the scraper will do so from the way the HTML is structured. So, if all your article pages are similar in that every `div` inside a `div` which comes after a `h1` is the article content, scrapers will get the article content based on that. Again, to break this, you can add / remove extra markup to your HTML, periodically and randomly, eg. adding extra `div`s or `span`s. With modern server side HTML processing, this should not be too hard.

Things to be aware of:

* It will be tedious and difficult to implement, maintain, and debug.

* You will hinder caching. Especially if you change ids or classes of your HTML elements, this will require corresponding changes in your CSS and JavaScript files, which means that every time you change them, they will have to be re-downloaded by the browser. This will result in longer page load times for repeat visitors, as well as increased server load. Although if you only change it once a week, it will not be a big problem.

* Clever scrapers will still be able to get your content by inferring where the actual content is, eg. by knowing that a large single block of text on the page is likely to be the actual article, or that a list of many similarly structured items is likely to be a search results page, and each item in the list is likely to contain the desired data. This makes it possible to still find & extract the desired data from the page. [Boilerpipe](https://code.google.com/archive/p/boilerpipe/) does exactly this.

Essentially, make sure that it is not easy for a script to find the actual, desired content for every similar page.

See also [How to prevent crawlers depending on XPath from getting page contents](http://stackoverflow.com/questions/30361740/) for details on how this can be implemented in PHP.


###Change your HTML based on the user's location

This is sort of similar to the previous tip. If you serve different HTML based on your user's location / country (determined by IP address), this may break scrapers which are delivered to users. For example, if someone is writing a mobile app which scrapes data from your site, it will work fine initially, but break when it's actually distributed to users, as those users may be in a different country, and thus get different HTML, which the embedded scraper was not designed to consume.

###Frequently change your HTML, actively screw with the scrapers by doing so !

An example: You have a search feature on your website, located at `example.com/search?query=somesearchquery`, which returns the following HTML:

    <div class="search-result">
      <h3 class="search-result-title">Stack Overflow has become the world's most popular programming Q & A website</h3>
      <p class="search-result-excerpt">The website Stack Overflow has now become the most popular programming Q & A website, with 10 million questions and many users, which...</p>
      <a class"search-result-link" href="/stories/stack-overflow-has-become-the-most-popular">Read more</a>
    </div>
    (And so on, lots more identically structured divs with search results)

As you may have guessed this is easy to scrape: all a scraper needs to do is hit the search URL with a query, and extract the desired data from the returned HTML. In addition to periodically changing the HTML as described above, you could also **leave the old markup with the old ids and classes in, hide it with CSS, and fill it with fake data, thereby poisoning the scraper.** Here's how the search results page could be changed:

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

This will mean that scrapers written to extract data from the HTML based on classes or IDs will continue to seemingly work, but they will get fake data or even ads, data which real users will never see, as they're hidden with CSS.

###Screw with the scraper: Insert fake, invisible honeypot data into your page

Adding on to the previous example, you can add invisible honeypot items to your HTML to catch scrapers. An example which could be added to the previously described search results page:

    <div class="search-result" style=”display:none">
      <h3 class="search-result-title">This search result is here to prevent scraping</h3>
      <p class="search-result-excerpt">If you're a human and see this, please ignore it. If you're a scraper, please click the link below :-)
      Note that clicking the link below will block access to this site for 24 hours.</p>
      <a class"search-result-link" href="/scrapertrap/scrapertrap.php">I'm a scraper !</a>
    </div>
    (The actual, real, search results follow.)

A scraper written to get all the search results will pick this up, just like any of the other, real search results on the page ..and visit the link, looking for the desired content. A real human  will never even see it in the first place (due to it being hidden with CSS), and won't visit the link. A genuine and desirable spider such as Google's will not visit the link either because you disallowed `/scrapertrap/` in your robots.txt (don't forget this!)

You can make your `scrapertrap.php` do something like block access for the IP address that visited it, force a captcha for all subsequent requests from that IP, create a loop of infinite redirects, or whatever.

* Don't forget to disallow your honeypot (`/scrapertrap/`) in your robots.txt file so that search engine bots don't fall into it.

* You can / should combine this with the previous tip of changing your HTML frequently.

* Change this frequently too, as scrapers will eventually learn to avoid it. Change the honeypot URL, and the text. You might also want to consider changing the inline CSS used for hiding, and use an ID attribute and external CSS instead, as scrapers will learn to avoid anything which has a `style` attribute with CSS used to hide the content. You might also want to try only enabling it sometimes, so the scraper works initially, but breaks after a while. This also applies to the previous tip.

###Serve fake and useless data if you detect a scraper

Again adding on to the previous items, if you detect what is obviously a scraper, you can serve up fake and useless data; this will corrupt the data the scraper gets from your website. You should also make it impossible to distinguish such fake data from real data, so that scrapers don't know that they're being screwed with.

As an example: if you have a news website; if you detect a scraper, instead of blocking access, just serve up fake, [randomly generated](https://en.wikipedia.org/wiki/Markov_chain#Markov_text_generators) articles, and this will poison the data the scraper gets. If you make your faked data or articles indistinguishable from the real thing, you'll make it hard for scrapers to get what they want, namely the actual, real articles. 

###Don't accept requests if the User Agent is empty / missing

Often, lazily written scrapers will not send a User Agent header with their request, whereas all  browsers as well as search engine spiders will. 

If you get a request where the User Agent header is not present, you can show a captcha, or simply block or limit access. (Or serve fake data as described above, or something else..)

Of course, this would is trivial to circumvent by sending a valid user agent string used by browsers, but as an additional measure against poorly written scrapers it is worth implementing.

###Don't accept requests if the User Agent is a common scraper one; blacklist ones used by scrapers

In some cases, scrapers will use a User Agent which no real browser or search engine spider uses, such as:

*  "Mozilla" (Just that, nothing else. I've seen a few questions about scraping here, using that. A real browser will never use only that single word.)
*  "Java 1.7.43_u43" (By default, Java's HttpUrlConnection uses something like this.)
*  "BIZCO EasyScraping Studio 2.0"
*  "wget", "curl", "libcurl",.. (Wget and cURL are sometimes used for basic scraping)

If you find that a specific User Agent string is used by scrapers on your site, and it is not used by real browsers or legitimate spiders, you can also add it to your blacklist.

Again, you could block / limit / show a captcha for requests with such User Agents. Again, this is easy to circumvent, and if there are real users using such a user agent, they will be affected too,

###Check the Referer header

Adding on to the previous item, you can also check for the [Referer](https://en.wikipedia.org/wiki/HTTP_referer header) (yes, it's Referer, not Referrer), as lazily written scrapers may not send it, or always send the same thing (sometimes "google.com"). As an example, if the user comes to an article page from a on-site search results page, check that the Referer header is present and points to that search results page.

Beware that:

* Real browsers don't always send it either;

* It's trivial to spoof.

Again, as an additional measure against poorly written scrapers it may be worth implementing.

###If it doesn't request assets (CSS, images), it's not a real browser.

A real browser will (almost always) request and download assets such as images and CSS. HTML parsers and scrapers won't as they are only interested in the actual pages and their content.

You could log requests to your assets, and if you see lots of requests for only the HTML, block, slow down access, or show a captcha.

Beware that:

* Search engine bots and legitimate spiders may not request your CSS, images, or other assets either.

* Screen readers, text-only browsers, ancient mobile browsers, and misconfigured devices may not request your assets.

* If the user has turned off images or CSS styling, their browser will not request these.

###Use and require cookies; use them to track user and scraper actions.

You can require cookies to be enabled in order to view your website. This will deter inexperienced and newbie scraper writers, however it is easy to for a scraper to send cookies. If you do use and require them, you can track user and scraper actions with them, and thus implement rate-limiting, blocking, or showing captchas on a per-user instead of a per-IP basis.

For example: when the user performs search, set a unique identifying cookie. When the results pages are viewed, verify that cookie. If the user opens all the search results (you can tell from the cookie), then it's probably a scraper.

Using cookies may be ineffective, as scrapers can send the cookies with their requests too, and discard them as needed. You will also prevent access for real users who have cookies disabled, if your site only works with cookies enabled.

Note that if you use JavaScript to set and retrieve the cookie, you'll block scrapers which don't run JavaScript, since they can't retrieve and send the cookie with their request.

###Use JavaScript + Ajax to load your content

You could use JavaScript + AJAX to load your content after the page itself loads. This will make the content inaccessible to HTML parsers which do not run JavaScript. This is often an effective deterrent to newbie and inexperienced programmers writing scrapers.

Things to be aware of:

* Using JavaScript to load the actual content will degrade user experience and performance (Who wants to wait for a search results page to load and then wait again for the content to be fetched and loaded ?).

* Search engines may not run JavaScript either (I'm not quite sure about this), thus preventing them from indexing your content. This may not be a problem for search results pages, but may be for other things, such as article pages.

* A programmer writing a scraper who knows what they're doing can discover the endpoints where the content is loaded from and use them. See also the next section on obfuscating your endpoints and the data available from them.

###Obfuscate your markup, network requests from scripts, and everything else.

If you use Ajax and JavaScript to load your data, you can obfuscate the data which is transferred. As an example, you could encode your data on the server (with something as simple as base64 or more complex with multiple layers of obfuscation, bit-shifting, and maybe even encryption), and then decode and display it on the client, after fetching via Ajax. This will mean that someone inspecting network traffic will not immediately see how your page works and loads data, and it will be tougher for someone to directly request request data from your endpoints, as they will have to reverse-engineer your descrambling algorithm.

* If you do use Ajax for loading the data, you should make it hard to use the endpoints without loading the page first, eg by requiring some session key as a parameter, which you can embed in your JavaScript or your HTML.

* You can also embed your obfuscated data directly in the initial HTML page and use JavaScript to deobfuscate and display it, which would avoid the extra network requests. Doing this will make it significantly harder to extract the data using a HTML-only parser which does not run JavaScript, as the one writing the scraper will have to reverse engineer your JavaScript (which you should obfuscate too).

* You might want to change your obfuscation methods regularly, to break scrapers who have figured it out.

There are several disadvantages to doing something like this, though:

* It will be tedious and difficult to implement, maintain, and debug.

* **It will be ineffective against scrapers and screenscrapers which actually run JavaScript and then extract the data**. (Most simple HTML parsers don't run JavaScript though)

* It will make your site nonfunctional for real users if they have JavaScript disabled.

* Performance and page-load times will suffer.

##Spiders:

Spiders are different from HTML scrapers in that they do not only seek to extract specific information from specific parts of your site, but that they often follow all links on your pages and attempt to find all the pages on your site. A typical use case would be to copy or mirror an entire website.

In addition to the points made above, which are also effective against spiders, it's worth noting that:

* Honeypots (as described above) can be especially effective. Add a link somewhere in your pages, which is invisible to humans (hide it with CSS), and which will not be followed by legitimate spiders (eg. Googles's), as legitimate spiders will respect your robots.txt, while spiders seeking to rip off your site may not. When something visits that link, you can log the IP address and block access or show a captcha for subsequent requests. Beware that sometimes, even undesirable spiders will respect your robots.txt, so this may not be entirely effective. The [top voted answer](http://stackoverflow.com/a/3161695/) explains this very well.

* Since most simple spiders don't run JavaScript, if you require JavaScript in order to load your content, those spiders will not get access to the content. See also my point about JavaScript above.

##Screenscrapers and scraping services:

Unfortunately, since these actually load your site in a real browser, run JavaScript, and so on, and sometimes even take a screenshot of your page and use OCR to get the desired data, these can be difficult or impossible deal with. However, you should still use rate limits, captchas, and restrict access when you get too many requests from a single IP address.

In addition, if the scraper does directly extract data from your HTML, the above techniques for HTML scrapers are also effective. To prevent OCR, you can change your page layout and appearance every once in a while, bit that's both extremely tedious and extremely irritating for users.

##Non-Technical:

###Tell people not to scrape, and some will respect it

You should tell people not to scrape your site, eg. in your conditions or Terms Of Service. Some people will actually respect that, and not scrape data from your website without permission.

###Find a lawyer

They know how to deal with copyright infringement, and can send a cease-and-desist letter. The DMCA is also helpful in this regard.

This is the approach Stack Overflow and Stack Exchange uses.

###Make your data available, provide an API:

This may seem counterproductive, but you could make your data easily available and require attribution and a link back to your site. Maybe even charge $$$ for it..

Again, Stack Exchange provides an API, but with attribution required.

##Miscellaneous:

* Find a balance between usability for real users and scraper-proofness: Everything you do will impact user experience negatively in one way or another, so you will need to find compromises.

* Don't forget your mobile site and apps. If you have a mobile version of your site, beware that scrapers can also scrape that. If you have a mobile app, that can be screen scraped too, and network traffic can be inspected to figure out the REST endpoints it uses.

* If you serve a special version of your site for specific browsers, eg. a cut-down version for older versions of Internet Explorer, don't forget that scrapers can scrape that, too.

* Use these tips in combination, pick what works best for you.

* Scrapers can scrape other scrapers: If there is one website which shows content scraped from your website, other scrapers can scrape from that scraper's website.

##What's the most effective way ?

In my experience of writing scrapers and helping people to write scrapers here on SO, the most effective methods are **Changing the HTML markup frequently**, **Honeypots and fake data**, **Using JavaScript, AJAX, and Cookies**, and **Rate limiting and scraper detection and subsequent blocking.**

##Further reading:

* [Wikipedia's article on Web scraping](http://en.wikipedia.org/wiki/Web_scraping). Many details on the technologies involved and the different types of web scraper, general information on how webscraping is done, as well as a look at the legalities of scraping.

**Good luck on the perilous journey of protecting your content...**
e Captchas if you suspect that your website is being accessed by a scraper.

Captchas ("Completely Automated Test to Tell Computers and Humans apart") are very effective against stopping scrapers. Unfortunately, they are also very effective at irritating users. 

As such, they are useful when you suspect a possible scraper, and want to stop the scraping, without also blocking access in case it isn't a scraper but a real user. You might want to consider showing a captcha before allowing access to the content if:

* You have many requests coming from the same IP address, more than a real user could generate (remember that IP addresses are often shared between multiple users, as also described). Google does this, for example.

* You get requests or a series of requests which a real user would be unlikely to perform, such as performing a search (or multiple searches in a row) and then visiting all the result links.

* Assets such as CSS are not requested.

* Etc..

Things to be aware of when using Captchas:

* Don't roll your own, use something like Google's [reCaptcha](https://www.google.com/recaptcha/intro/index.html) : It's a lot easier than implementing a captcha yourself, it's more user-friendly than some blurry and warped text solution you might come up with yourself (users often only need to tick a box), and it's also a lot harder for a scripter to solve than a simple image served from your site

* Don't include the solution to the captcha in the HTML markup: I've actually seen one website which had the solution for the captcha _in the page itself_, (although quite well hidden) thus making it pretty useless. Don't do something like this. Again, use a service like reCaptcha, and you won't have this kind of problem (if you use it properly).

* Captchas can be solved in bulk: There are captcha-solving services where actual, low-paid, humans solve captchas in bulk. Again, using reCaptcha is a good idea here, as they have protections (such as the relatively short time the user has in order to solve the captcha). This kind of service is unlikely to by used unless your data is really valuable.

###Serve your text content as an image

You can render text into an image server-side, and serve that to be displayed, which will hinder simple scrapers extracting text.

 However, this is bad for screen readers, search engines, performance, and pretty much everything else. It's also illegal in some places (due to accessibility, eg. the Americans with Disabilities Act), and it's also easy to circumvent with some OCR, so don't do it. 

You can do something similar with CSS sprites, but that suffers from the same problems.

###Don't expose your complete dataset:

If feasible, don't provide a way for a script / bot to get all of your dataset. As an example: You have a news site, with lots of individual articles. You could make those articles be only accessible by searching for them via the on site search, and, if you don't have a list of _all_ the articles on the site and their URLs anywhere, those articles will be only accessible by using the search feature. This means that a script wanting to get all the articles off your site will have to do searches for all possible phrases which may appear in your articles in order to find them all, which will be time-consuming, horribly inefficient, and will hopefully make the scraper give up.

This will be ineffective if:

* The bot / script does not want / need the full dataset anyway.
* Your articles are served from a URL which looks something like `example.com/article.php?articleId=12345`. This (and similar things) which will allow scrapers to simply iterate over all the `articleId`s and request all the articles that way.
* There are other ways to eventually find all the articles, such as by writing a script to follow links within articles which lead to other articles.
* Searching for something like "and" or "the" can reveal almost everything, so that is something to be aware of. (You could / should avoid this by only returning the top 10 or 20 results).
* You need search engines to find your content.

An example of a site which does this is Google News.

###Don't expose your APIs, endpoints, and similar things:

Make sure you don't expose any APIs, even unintentionally. For example, if you are using AJAX or network requests from within Adobe Flash or Java Applets (God forbid!) to load your data it is trivial to look at the network requests from the page and figure out where those requests are going to, and then reverse engineer and use those endpoints in a scraper program. Make sure you obfuscate your endpoints and make them hard for others to use, as also described below.

##To deter HTML parsers and scrapers:

Since HTML parsers work by extracting content from pages based on identifiable patterns in the HTML, we can intentionally change those patterns in oder to break these scrapers, or even screw with them. Most of these tips also apply to other scrapers like spiders and screenscrapers too. Some ideas:

###Frequently change your HTML

Scrapers which process HTML directly do so by extracting contents from specific, identifiable parts of your HTML page. For example: If all pages on your website have a `div` with an id of `article-content`, which contains the text of the article, then it is trivial to write a script to visit all the article pages on your site, and extract the content text of the `article-content ` div on each article page, and voilà, the scraper has all the articles from your site in a format that can be reused elsewhere.

If you change the HTML and the structure of your pages frequently, such scrapers will no longer work.

* You can frequently change the id's and classes of elements in your HTML, perhaps even automatically. So, if your `div.article-content` becomes something like `div.a4c36dda13eaf0`, and changes every week, the scraper will work fine initially, but will break after a week. Make sure to change the length of your ids / classes too, otherwise the scraper will use `div.[any-14-characters]` to find the desired div instead. Beware of other similar holes too..

* If there is no way to find the desired content from the markup, the scraper will do so from the way the HTML is structured. So, if all your article pages are similar in that every `div` inside a `div` which comes after a `h1` is the article content, scrapers will get the article content based on that. Again, to break this, you can add / remove extra markup to your HTML, periodically and randomly, eg. adding extra `div`s or `span`s. With modern server side HTML processing, this should not be too hard.

Things to be aware of:

* It will be tedious and difficult to implement, maintain, and debug.

* You will hinder caching. Especially if you change ids or classes of your HTML elements, this will require corresponding changes in your CSS and JavaScript files, which means that every time you change them, they will have to be re-downloaded by the browser. This will result in longer page load times for repeat visitors, as well as increased server load. Although if you only change it once a week, it will not be a big problem.

* Clever scrapers will still be able to get your content by inferring where the actual content is, eg. by knowing that a large single block of text on the page is likely to be the actual article, or that a list of many similarly structured items is likely to be a search results page, and each item in the list is likely to contain the desired data. This makes it possible to still find & extract the desired data from the page. [Boilerpipe](https://code.google.com/archive/p/boilerpipe/) does exactly this.

Essentially, make sure that it is not easy for a script to find the actual, desired content for every similar page.

See also [How to prevent crawlers depending on XPath from getting page contents](http://stackoverflow.com/questions/30361740/) for details on how this can be implemented in PHP.


###Change your HTML based on the user's location

This is sort of similar to the previous tip. If you serve different HTML based on your user's location / country (determined by IP address), this may break scrapers which are delivered to users. For example, if someone is writing a mobile app which scrapes data from your site, it will work fine initially, but break when it's actually distributed to users, as those users may be in a different country, and thus get different HTML, which the embedded scraper was not designed to consume.

###Frequently change your HTML, actively screw with the scrapers by doing so !

An example: You have a search feature on your website, located at `example.com/search?query=somesearchquery`, which returns the following HTML:

    <div class="search-result">
      <h3 class="search-result-title">Stack Overflow has become the world's most popular programming Q & A website</h3>
      <p class="search-result-excerpt">The website Stack Overflow has now become the most popular programming Q & A website, with 10 million questions and many users, which...</p>
      <a class"search-result-link" href="/stories/stack-overflow-has-become-the-most-popular">Read more</a>
    </div>
    (And so on, lots more identically structured divs with search results)

As you may have guessed this is easy to scrape: all a scraper needs to do is hit the search URL with a query, and extract the desired data from the returned HTML. In addition to periodically changing the HTML as described above, you could also **leave the old markup with the old ids and classes in, hide it with CSS, and fill it with fake data, thereby poisoning the scraper.** Here's how the search results page could be changed:

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

This will mean that scrapers written to extract data from the HTML based on classes or IDs will continue to seemingly work, but they will get fake data or even ads, data which real users will never see, as they're hidden with CSS.

###Screw with the scraper: Insert fake, invisible honeypot data into your page

Adding on to the previous example, you can add invisible honeypot items to your HTML to catch scrapers. An example which could be added to the previously described search results page:

    <div class="search-result" style=”display:none">
      <h3 class="search-result-title">This search result is here to prevent scraping</h3>
      <p class="search-result-excerpt">If you're a human and see this, please ignore it. If you're a scraper, please click the link below :-)
      Note that clicking the link below will block access to this site for 24 hours.</p>
      <a class"search-result-link" href="/scrapertrap/scrapertrap.php">I'm a scraper !</a>
    </div>
    (The actual, real, search results follow.)

A scraper written to get all the search results will pick this up, just like any of the other, real search results on the page ..and visit the link, looking for the desired content. A real human  will never even see it in the first place (due to it being hidden with CSS), and won't visit the link. A genuine and desirable spider such as Google's will not visit the link either because you disallowed `/scrapertrap/` in your robots.txt (don't forget this!)

You can make your `scrapertrap.php` do something like block access for the IP address that visited it, force a captcha for all subsequent requests from that IP, create a loop of infinite redirects, or whatever.

* Don't forget to disallow your honeypot (`/scrapertrap/`) in your robots.txt file so that search engine bots don't fall into it.

* You can / should combine this with the previous tip of changing your HTML frequently.

* Change this frequently too, as scrapers will eventually learn to avoid it. Change the honeypot URL, and the text. You might also want to consider changing the inline CSS used for hiding, and use an ID attribute and external CSS instead, as scrapers will learn to avoid anything which has a `style` attribute with CSS used to hide the content. You might also want to try only enabling it sometimes, so the scraper works initially, but breaks after a while. This also applies to the previous tip.

###Serve fake and useless data if you detect a scraper

Again adding on to the previous items, if you detect what is obviously a scraper, you can serve up fake and useless data; this will corrupt the data the scraper gets from your website. You should also make it impossible to distinguish such fake data from real data, so that scrapers don't know that they're being screwed with.

As an example: if you have a news website; if you detect a scraper, instead of blocking access, just serve up fake, [randomly generated](https://en.wikipedia.org/wiki/Markov_chain#Markov_text_generators) articles, and this will poison the data the scraper gets. If you make your faked data or articles indistinguishable from the real thing, you'll make it hard for scrapers to get what they want, namely the actual, real articles. 

###Don't accept requests if the User Agent is empty / missing

Often, lazily written scrapers will not send a User Agent header with their request, whereas all  browsers as well as search engine spiders will. 

If you get a request where the User Agent header is not present, you can show a captcha, or simply block or limit access. (Or serve fake data as described above, or something else..)

Of course, this would is trivial to circumvent by sending a valid user agent string used by browsers, but as an additional measure against poorly written scrapers it is worth implementing.

###Don't accept requests if the User Agent is a common scraper one; blacklist ones used by scrapers

In some cases, scrapers will use a User Agent which no real browser or search engine spider uses, such as:

*  "Mozilla" (Just that, nothing else. I've seen a few questions about scraping here, using that. A real browser will never use only that single word.)
*  "Java 1.7.43_u43" (By default, Java's HttpUrlConnection uses something like this.)
*  "BIZCO EasyScraping Studio 2.0"
*  "wget", "curl", "libcurl",.. (Wget and cURL are sometimes used for basic scraping)

If you find that a specific User Agent string is used by scrapers on your site, and it is not used by real browsers or legitimate spiders, you can also add it to your blacklist.

Again, you could block / limit / show a captcha for requests with such User Agents. Again, this is easy to circumvent, and if there are real users using such a user agent, they will be affected too,

###Check the Referer header

Adding on to the previous item, you can also check for the [Referer](https://en.wikipedia.org/wiki/HTTP_referer header) (yes, it's Referer, not Referrer), as lazily written scrapers may not send it, or always send the same thing (sometimes "google.com"). As an example, if the user comes to an article page from a on-site search results page, check that the Referer header is present and points to that search results page.

Beware that:

* Real browsers don't always send it either;

* It's trivial to spoof.

Again, as an additional measure against poorly written scrapers it may be worth implementing.

###If it doesn't request assets (CSS, images), it's not a real browser.

A real browser will (almost always) request and download assets such as images and CSS. HTML parsers and scrapers won't as they are only interested in the actual pages and their content.

You could log requests to your assets, and if you see lots of requests for only the HTML, block, slow down access, or show a captcha.

Beware that:

* Search engine bots and legitimate spiders may not request your CSS, images, or other assets either.

* Screen readers, text-only browsers, ancient mobile browsers, and misconfigured devices may not request your assets.

* If the user has turned off images or CSS styling, their browser will not request these.

###Use and require cookies; use them to track user and scraper actions.

You can require cookies to be enabled in order to view your website. This will deter inexperienced and newbie scraper writers, however it is easy to for a scraper to send cookies. If you do use and require them, you can track user and scraper actions with them, and thus implement rate-limiting, blocking, or showing captchas on a per-user instead of a per-IP basis.

For example: when the user performs search, set a unique identifying cookie. When the results pages are viewed, verify that cookie. If the user opens all the search results (you can tell from the cookie), then it's probably a scraper.

Using cookies may be ineffective, as scrapers can send the cookies with their requests too, and discard them as needed. You will also prevent access for real users who have cookies disabled, if your site only works with cookies enabled.

Note that if you use JavaScript to set and retrieve the cookie, you'll block scrapers which don't run JavaScript, since they can't retrieve and send the cookie with their request.

###Use JavaScript + Ajax to load your content

You could use JavaScript + AJAX to load your content after the page itself loads. This will make the content inaccessible to HTML parsers which do not run JavaScript. This is often an effective deterrent to newbie and inexperienced programmers writing scrapers.

Things to be aware of:

* Using JavaScript to load the actual content will degrade user experience and performance (Who wants to wait for a search results page to load and then wait again for the content to be fetched and loaded ?).

* Search engines may not run JavaScript either (I'm not quite sure about this), thus preventing them from indexing your content. This may not be a problem for search results pages, but may be for other things, such as article pages.

* A programmer writing a scraper who knows what they're doing can discover the endpoints where the content is loaded from and use them. See also the next section on obfuscating your endpoints and the data available from them.

###Obfuscate your markup, network requests from scripts, and everything else.

If you use Ajax and JavaScript to load your data, you can obfuscate the data which is transferred. As an example, you could encode your data on the server (with something as simple as base64 or more complex with multiple layers of obfuscation, bit-shifting, and maybe even encryption), and then decode and display it on the client, after fetching via Ajax. This will mean that someone inspecting network traffic will not immediately see how your page works and loads data, and it will be tougher for someone to directly request request data from your endpoints, as they will have to reverse-engineer your descrambling algorithm.

* If you do use Ajax for loading the data, you should make it hard to use the endpoints without loading the page first, eg by requiring some session key as a parameter, which you can embed in your JavaScript or your HTML.

* You can also embed your obfuscated data directly in the initial HTML page and use JavaScript to deobfuscate and display it, which would avoid the extra network requests. Doing this will make it significantly harder to extract the data using a HTML-only parser which does not run JavaScript, as the one writing the scraper will have to reverse engineer your JavaScript (which you should obfuscate too).

* You might want to change your obfuscation methods regularly, to break scrapers who have figured it out.

There are several disadvantages to doing something like this, though:

* It will be tedious and difficult to implement, maintain, and debug.

* **It will be ineffective against scrapers and screenscrapers which actually run JavaScript and then extract the data**. (Most simple HTML parsers don't run JavaScript though)

* It will make your site nonfunctional for real users if they have JavaScript disabled.

* Performance and page-load times will suffer.

##Spiders:

Spiders are different from HTML scrapers in that they do not only seek to extract specific information from specific parts of your site, but that they often follow all links on your pages and attempt to find all the pages on your site. A typical use case would be to copy or mirror an entire website.

In addition to the points made above, which are also effective against spiders, it's worth noting that:

* Honeypots (as described above) can be especially effective. Add a link somewhere in your pages, which is invisible to humans (hide it with CSS), and which will not be followed by legitimate spiders (eg. Googles's), as legitimate spiders will respect your robots.txt, while spiders seeking to rip off your site may not. When something visits that link, you can log the IP address and block access or show a captcha for subsequent requests. Beware that sometimes, even undesirable spiders will respect your robots.txt, so this may not be entirely effective. The [top voted answer](http://stackoverflow.com/a/3161695/) explains this very well.

* Since most simple spiders don't run JavaScript, if you require JavaScript in order to load your content, those spiders will not get access to the content. See also my point about JavaScript above.

##Screenscrapers and scraping services:

Unfortunately, since these actually load your site in a real browser, run JavaScript, and so on, and sometimes even take a screenshot of your page and use OCR to get the desired data, these can be difficult or impossible deal with. However, you should still use rate limits, captchas, and restrict access when you get too many requests from a single IP address.

In addition, if the scraper does directly extract data from your HTML, the above techniques for HTML scrapers are also effective. To prevent OCR, you can change your page layout and appearance every once in a while, bit that's both extremely tedious and extremely irritating for users.

##Non-Technical:

###Tell people not to scrape, and some will respect it

You should tell people not to scrape your site, eg. in your conditions or Terms Of Service. Some people will actually respect that, and not scrape data from your website without permission.

###Find a lawyer

They know how to deal with copyright infringement, and can send a cease-and-desist letter. The DMCA is also helpful in this regard.

This is the approach Stack Overflow and Stack Exchange uses.

###Make your data available, provide an API:

This may seem counterproductive, but you could make your data easily available and require attribution and a link back to your site. Maybe even charge $$$ for it..

Again, Stack Exchange provides an API, but with attribution required.

##Miscellaneous:

* Find a balance between usability for real users and scraper-proofness: Everything you do will impact user experience negatively in one way or another, so you will need to find compromises.

* Don't forget your mobile site and apps. If you have a mobile version of your site, beware that scrapers can also scrape that. If you have a mobile app, that can be screen scraped too, and network traffic can be inspected to figure out the REST endpoints it uses.

* If you serve a special version of your site for specific browsers, eg. a cut-down version for older versions of Internet Explorer, don't forget that scrapers can scrape that, too.

* Use these tips in combination, pick what works best for you.

* Scrapers can scrape other scrapers: If there is one website which shows content scraped from your website, other scrapers can scrape from that scraper's website.

##What's the most effective way ?

In my experience of writing scrapers and helping people to write scrapers here on SO, the most effective methods are **Changing the HTML markup frequently**, **Honeypots and fake data**, **Using JavaScript, AJAX, and Cookies**, and **Rate limiting and scraper detection and subsequent blocking.**

##Further reading:

* [Wikipedia's article on Web scraping](http://en.wikipedia.org/wiki/Web_scraping). Many details on the technologies involved and the different types of web scraper, general information on how webscraping is done, as well as a look at the legalities of scraping.

**Good luck on the perilous journey of protecting your content...**
real user. You might want to consider showing a captcha before allowing access to the content if:

* You have many requests coming from the same IP address, more than a real user could generate (remember that IP addresses are often shared between multiple users, as also described). Google does this, for example.

* You get requests or a series of requests which a real user would be unlikely to perform, such as performing a search (or multiple searches in a row) and then visiting all the result links.

* Assets such as CSS are not requested.

* Etc..

Things to be aware of when using Captchas:

* Don't roll your own, use something like Google's [reCaptcha](https://www.google.com/recaptcha/intro/index.html) : It's a lot easier than implementing a captcha yourself, it's more user-friendly than some blurry and warped text solution you might come up with yourself (users often only need to tick a box), and it's also a lot harder for a scripter to solve than a simple image served from your site

* Don't include the solution to the captcha in the HTML markup: I've actually seen one website which had the solution for the captcha _in the page itself_, (although quite well hidden) thus making it pretty useless. Don't do something like this. Again, use a service like reCaptcha, and you won't have this kind of problem (if you use it properly).

* Captchas can be solved in bulk: There are captcha-solving services where actual, low-paid, humans solve captchas in bulk. Again, using reCaptcha is a good idea here, as they have protections (such as the relatively short time the user has in order to solve the captcha). This kind of service is unlikely to by used unless your data is really valuable.

###Serve your text content as an image

You can render text into an image server-side, and serve that to be displayed, which will hinder simple scrapers extracting text.

 However, this is bad for screen readers, search engines, performance, and pretty much everything else. It's also illegal in some places (due to accessibility, eg. the Americans with Disabilities Act), and it's also easy to circumvent with some OCR, so don't do it. 

You can do something similar with CSS sprites, but that suffers from the same problems.

###Don't expose your complete dataset:

If feasible, don't provide a way for a script / bot to get all of your dataset. As an example: You have a news site, with lots of individual articles. You could make those articles be only accessible by searching for them via the on site search, and, if you don't have a list of _all_ the articles on the site and their URLs anywhere, those articles will be only accessible by using the search feature. This means that a script wanting to get all the articles off your site will have to do searches for all possible phrases which may appear in your articles in order to find them all, which will be time-consuming, horribly inefficient, and will hopefully make the scraper give up.

This will be ineffective if:

* The bot / script does not want / need the full dataset anyway.
* Your articles are served from a URL which looks something like `example.com/article.php?articleId=12345`. This (and similar things) which will allow scrapers to simply iterate over all the `articleId`s and request all the articles that way.
* There are other ways to eventually find all the articles, such as by writing a script to follow links within articles which lead to other articles.
* Searching for something like "and" or "the" can reveal almost everything, so that is something to be aware of. (You could / should avoid this by only returning the top 10 or 20 results).
* You need search engines to find your content.

An example of a site which does this is Google News.

###Don't expose your APIs, endpoints, and similar things:

Make sure you don't expose any APIs, even unintentionally. For example, if you are using AJAX or network requests from within Adobe Flash or Java Applets (God forbid!) to load your data it is trivial to look at the network requests from the page and figure out where those requests are going to, and then reverse engineer and use those endpoints in a scraper program. Make sure you obfuscate your endpoints and make them hard for others to use, as also described below.

##To deter HTML parsers and scrapers:

Since HTML parsers work by extracting content from pages based on identifiable patterns in the HTML, we can intentionally change those patterns in oder to break these scrapers, or even screw with them. Most of these tips also apply to other scrapers like spiders and screenscrapers too. Some ideas:

###Frequently change your HTML

Scrapers which process HTML directly do so by extracting contents from specific, identifiable parts of your HTML page. For example: If all pages on your website have a `div` with an id of `article-content`, which contains the text of the article, then it is trivial to write a script to visit all the article pages on your site, and extract the content text of the `article-content ` div on each article page, and voilà, the scraper has all the articles from your site in a format that can be reused elsewhere.

If you change the HTML and the structure of your pages frequently, such scrapers will no longer work.

* You can frequently change the id's and classes of elements in your HTML, perhaps even automatically. So, if your `div.article-content` becomes something like `div.a4c36dda13eaf0`, and changes every week, the scraper will work fine initially, but will break after a week. Make sure to change the length of your ids / classes too, otherwise the scraper will use `div.[any-14-characters]` to find the desired div instead. Beware of other similar holes too..

* If there is no way to find the desired content from the markup, the scraper will do so from the way the HTML is structured. So, if all your article pages are similar in that every `div` inside a `div` which comes after a `h1` is the article content, scrapers will get the article content based on that. Again, to break this, you can add / remove extra markup to your HTML, periodically and randomly, eg. adding extra `div`s or `span`s. With modern server side HTML processing, this should not be too hard.

Things to be aware of:

* It will be tedious and difficult to implement, maintain, and debug.

* You will hinder caching. Especially if you change ids or classes of your HTML elements, this will require corresponding changes in your CSS and JavaScript files, which means that every time you change them, they will have to be re-downloaded by the browser. This will result in longer page load times for repeat visitors, as well as increased server load. Although if you only change it once a week, it will not be a big problem.

* Clever scrapers will still be able to get your content by inferring where the actual content is, eg. by knowing that a large single block of text on the page is likely to be the actual article, or that a list of many similarly structured items is likely to be a search results page, and each item in the list is likely to contain the desired data. This makes it possible to still find & extract the desired data from the page. [Boilerpipe](https://code.google.com/archive/p/boilerpipe/) does exactly this.

Essentially, make sure that it is not easy for a script to find the actual, desired content for every similar page.

See also [How to prevent crawlers depending on XPath from getting page contents](http://stackoverflow.com/questions/30361740/) for details on how this can be implemented in PHP.


###Change your HTML based on the user's location

This is sort of similar to the previous tip. If you serve different HTML based on your user's location / country (determined by IP address), this may break scrapers which are delivered to users. For example, if someone is writing a mobile app which scrapes data from your site, it will work fine initially, but break when it's actually distributed to users, as those users may be in a different country, and thus get different HTML, which the embedded scraper was not designed to consume.

###Frequently change your HTML, actively screw with the scrapers by doing so !

An example: You have a search feature on your website, located at `example.com/search?query=somesearchquery`, which returns the following HTML:

    <div class="search-result">
      <h3 class="search-result-title">Stack Overflow has become the world's most popular programming Q & A website</h3>
      <p class="search-result-excerpt">The website Stack Overflow has now become the most popular programming Q & A website, with 10 million questions and many users, which...</p>
      <a class"search-result-link" href="/stories/stack-overflow-has-become-the-most-popular">Read more</a>
    </div>
    (And so on, lots more identically structured divs with search results)

As you may have guessed this is easy to scrape: all a scraper needs to do is hit the search URL with a query, and extract the desired data from the returned HTML. In addition to periodically changing the HTML as described above, you could also **leave the old markup with the old ids and classes in, hide it with CSS, and fill it with fake data, thereby poisoning the scraper.** Here's how the search results page could be changed:

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

This will mean that scrapers written to extract data from the HTML based on classes or IDs will continue to seemingly work, but they will get fake data or even ads, data which real users will never see, as they're hidden with CSS.

###Screw with the scraper: Insert fake, invisible honeypot data into your page

Adding on to the previous example, you can add invisible honeypot items to your HTML to catch scrapers. An example which could be added to the previously described search results page:

    <div class="search-result" style=”display:none">
      <h3 class="search-result-title">This search result is here to prevent scraping</h3>
      <p class="search-result-excerpt">If you're a human and see this, please ignore it. If you're a scraper, please click the link below :-)
      Note that clicking the link below will block access to this site for 24 hours.</p>
      <a class"search-result-link" href="/scrapertrap/scrapertrap.php">I'm a scraper !</a>
    </div>
    (The actual, real, search results follow.)

A scraper written to get all the search results will pick this up, just like any of the other, real search results on the page ..and visit the link, looking for the desired content. A real human  will never even see it in the first place (due to it being hidden with CSS), and won't visit the link. A genuine and desirable spider such as Google's will not visit the link either because you disallowed `/scrapertrap/` in your robots.txt (don't forget this!)

You can make your `scrapertrap.php` do something like block access for the IP address that visited it, force a captcha for all subsequent requests from that IP, create a loop of infinite redirects, or whatever.

* Don't forget to disallow your honeypot (`/scrapertrap/`) in your robots.txt file so that search engine bots don't fall into it.

* You can / should combine this with the previous tip of changing your HTML frequently.

* Change this frequently too, as scrapers will eventually learn to avoid it. Change the honeypot URL, and the text. You might also want to consider changing the inline CSS used for hiding, and use an ID attribute and external CSS instead, as scrapers will learn to avoid anything which has a `style` attribute with CSS used to hide the content. You might also want to try only enabling it sometimes, so the scraper works initially, but breaks after a while. This also applies to the previous tip.

###Serve fake and useless data if you detect a scraper

Again adding on to the previous items, if you detect what is obviously a scraper, you can serve up fake and useless data; this will corrupt the data the scraper gets from your website. You should also make it impossible to distinguish such fake data from real data, so that scrapers don't know that they're being screwed with.

As an example: if you have a news website; if you detect a scraper, instead of blocking access, just serve up fake, [randomly generated](https://en.wikipedia.org/wiki/Markov_chain#Markov_text_generators) articles, and this will poison the data the scraper gets. If you make your faked data or articles indistinguishable from the real thing, you'll make it hard for scrapers to get what they want, namely the actual, real articles. 

###Don't accept requests if the User Agent is empty / missing

Often, lazily written scrapers will not send a User Agent header with their request, whereas all  browsers as well as search engine spiders will. 

If you get a request where the User Agent header is not present, you can show a captcha, or simply block or limit access. (Or serve fake data as described above, or something else..)

Of course, this would is trivial to circumvent by sending a valid user agent string used by browsers, but as an additional measure against poorly written scrapers it is worth implementing.

###Don't accept requests if the User Agent is a common scraper one; blacklist ones used by scrapers

In some cases, scrapers will use a User Agent which no real browser or search engine spider uses, such as:

*  "Mozilla" (Just that, nothing else. I've seen a few questions about scraping here, using that. A real browser will never use only that single word.)
*  "Java 1.7.43_u43" (By default, Java's HttpUrlConnection uses something like this.)
*  "BIZCO EasyScraping Studio 2.0"
*  "wget", "curl", "libcurl",.. (Wget and cURL are sometimes used for basic scraping)

If you find that a specific User Agent string is used by scrapers on your site, and it is not used by real browsers or legitimate spiders, you can also add it to your blacklist.

Again, you could block / limit / show a captcha for requests with such User Agents. Again, this is easy to circumvent, and if there are real users using such a user agent, they will be affected too,

###Check the Referer header

Adding on to the previous item, you can also check for the [Referer](https://en.wikipedia.org/wiki/HTTP_referer header) (yes, it's Referer, not Referrer), as lazily written scrapers may not send it, or always send the same thing (sometimes "google.com"). As an example, if the user comes to an article page from a on-site search results page, check that the Referer header is present and points to that search results page.

Beware that:

* Real browsers don't always send it either;

* It's trivial to spoof.

Again, as an additional measure against poorly written scrapers it may be worth implementing.

###If it doesn't request assets (CSS, images), it's not a real browser.

A real browser will (almost always) request and download assets such as images and CSS. HTML parsers and scrapers won't as they are only interested in the actual pages and their content.

You could log requests to your assets, and if you see lots of requests for only the HTML, block, slow down access, or show a captcha.

Beware that:

* Search engine bots and legitimate spiders may not request your CSS, images, or other assets either.

* Screen readers, text-only browsers, ancient mobile browsers, and misconfigured devices may not request your assets.

* If the user has turned off images or CSS styling, their browser will not request these.

###Use and require cookies; use them to track user and scraper actions.

You can require cookies to be enabled in order to view your website. This will deter inexperienced and newbie scraper writers, however it is easy to for a scraper to send cookies. If you do use and require them, you can track user and scraper actions with them, and thus implement rate-limiting, blocking, or showing captchas on a per-user instead of a per-IP basis.

For example: when the user performs search, set a unique identifying cookie. When the results pages are viewed, verify that cookie. If the user opens all the search results (you can tell from the cookie), then it's probably a scraper.

Using cookies may be ineffective, as scrapers can send the cookies with their requests too, and discard them as needed. You will also prevent access for real users who have cookies disabled, if your site only works with cookies enabled.

Note that if you use JavaScript to set and retrieve the cookie, you'll block scrapers which don't run JavaScript, since they can't retrieve and send the cookie with their request.

###Use JavaScript + Ajax to load your content

You could use JavaScript + AJAX to load your content after the page itself loads. This will make the content inaccessible to HTML parsers which do not run JavaScript. This is often an effective deterrent to newbie and inexperienced programmers writing scrapers.

Things to be aware of:

* Using JavaScript to load the actual content will degrade user experience and performance (Who wants to wait for a search results page to load and then wait again for the content to be fetched and loaded ?).

* Search engines may not run JavaScript either (I'm not quite sure about this), thus preventing them from indexing your content. This may not be a problem for search results pages, but may be for other things, such as article pages.

* A programmer writing a scraper who knows what they're doing can discover the endpoints where the content is loaded from and use them. See also the next section on obfuscating your endpoints and the data available from them.

###Obfuscate your markup, network requests from scripts, and everything else.

If you use Ajax and JavaScript to load your data, you can obfuscate the data which is transferred. As an example, you could encode your data on the server (with something as simple as base64 or more complex with multiple layers of obfuscation, bit-shifting, and maybe even encryption), and then decode and display it on the client, after fetching via Ajax. This will mean that someone inspecting network traffic will not immediately see how your page works and loads data, and it will be tougher for someone to directly request request data from your endpoints, as they will have to reverse-engineer your descrambling algorithm.

* If you do use Ajax for loading the data, you should make it hard to use the endpoints without loading the page first, eg by requiring some session key as a parameter, which you can embed in your JavaScript or your HTML.

* You can also embed your obfuscated data directly in the initial HTML page and use JavaScript to deobfuscate and display it, which would avoid the extra network requests. Doing this will make it significantly harder to extract the data using a HTML-only parser which does not run JavaScript, as the one writing the scraper will have to reverse engineer your JavaScript (which you should obfuscate too).

* You might want to change your obfuscation methods regularly, to break scrapers who have figured it out.

There are several disadvantages to doing something like this, though:

* It will be tedious and difficult to implement, maintain, and debug.

* **It will be ineffective against scrapers and screenscrapers which actually run JavaScript and then extract the data**. (Most simple HTML parsers don't run JavaScript though)

* It will make your site nonfunctional for real users if they have JavaScript disabled.

* Performance and page-load times will suffer.

##Spiders:

Spiders are different from HTML scrapers in that they do not only seek to extract specific information from specific parts of your site, but that they often follow all links on your pages and attempt to find all the pages on your site. A typical use case would be to copy or mirror an entire website.

In addition to the points made above, which are also effective against spiders, it's worth noting that:

* Honeypots (as described above) can be especially effective. Add a link somewhere in your pages, which is invisible to humans (hide it with CSS), and which will not be followed by legitimate spiders (eg. Googles's), as legitimate spiders will respect your robots.txt, while spiders seeking to rip off your site may not. When something visits that link, you can log the IP address and block access or show a captcha for subsequent requests. Beware that sometimes, even undesirable spiders will respect your robots.txt, so this may not be entirely effective. The [top voted answer](http://stackoverflow.com/a/3161695/) explains this very well.

* Since most simple spiders don't run JavaScript, if you require JavaScript in order to load your content, those spiders will not get access to the content. See also my point about JavaScript above.

##Screenscrapers and scraping services:

Unfortunately, since these actually load your site in a real browser, run JavaScript, and so on, and sometimes even take a screenshot of your page and use OCR to get the desired data, these can be difficult or impossible deal with. However, you should still use rate limits, captchas, and restrict access when you get too many requests from a single IP address.

In addition, if the scraper does directly extract data from your HTML, the above techniques for HTML scrapers are also effective. To prevent OCR, you can change your page layout and appearance every once in a while, bit that's both extremely tedious and extremely irritating for users.

##Non-Technical:

###Tell people not to scrape, and some will respect it

You should tell people not to scrape your site, eg. in your conditions or Terms Of Service. Some people will actually respect that, and not scrape data from your website without permission.

###Find a lawyer

They know how to deal with copyright infringement, and can send a cease-and-desist letter. The DMCA is also helpful in this regard.

This is the approach Stack Overflow and Stack Exchange uses.

###Make your data available, provide an API:

This may seem counterproductive, but you could make your data easily available and require attribution and a link back to your site. Maybe even charge $$$ for it..

Again, Stack Exchange provides an API, but with attribution required.

##Miscellaneous:

* Find a balance between usability for real users and scraper-proofness: Everything you do will impact user experience negatively in one way or another, so you will need to find compromises.

* Don't forget your mobile site and apps. If you have a mobile version of your site, beware that scrapers can also scrape that. If you have a mobile app, that can be screen scraped too, and network traffic can be inspected to figure out the REST endpoints it uses.

* If you serve a special version of your site for specific browsers, eg. a cut-down version for older versions of Internet Explorer, don't forget that scrapers can scrape that, too.

* Use these tips in combination, pick what works best for you.

* Scrapers can scrape other scrapers: If there is one website which shows content scraped from your website, other scrapers can scrape from that scraper's website.

##What's the most effective way ?

In my experience of writing scrapers and helping people to write scrapers here on SO, the most effective methods are **Changing the HTML markup frequently**, **Honeypots and fake data**, **Using JavaScript, AJAX, and Cookies**, and **Rate limiting and scraper detection and subsequent blocking.**

##Further reading:

* [Wikipedia's article on Web scraping](http://en.wikipedia.org/wiki/Web_scraping). Many details on the technologies involved and the different types of web scraper, general information on how webscraping is done, as well as a look at the legalities of scraping.

**Good luck on the perilous journey of protecting your content...**
