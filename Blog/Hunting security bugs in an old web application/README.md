# [Hunting security bugs in an old web application](https://www.abhith.net/post/hunting-security-bugs-in-an-old-web-application/)
## Post Attributes
### Tags
Security, Bug, Code-Review 
### Categories
ASP.NET 
### Excerpt
In this post, I am sharing one of my security bug hunting experience in an older ASP.NET web form project. 
### Published Date
2017-10-09 09:36:25
## Content
### Markdown
Recently one of our old (~4 years or so) website faced a big security challenge. A similar situation we faced for the same website a long time ago when it initially deployed to the production server, that time the site was hosted in **[GoDaddy][1]**. Due to that incident, we moved the site to a custom VPS server with a third party company (where I worked during that time) and everything was fine after that.

This year we moved the website to **[Everleap][2]** and then recently the same problem happened, but after it happened, we brought the site back online within 5 minutes with the help of Scheduled database backups and file backup that were configured for the same (which we do for all the sites we manage). **[Everleap][2]** have a cloud backup service which will take care of the periodic backup of files as well as the databases.

The site was configured to use **[Sucuri][3]** firewall but the portal's access was restricted only to the client. And IP security was enabled for the site, so that only **[Sucuri][3]** IP's are allowed to access the site. The site content was managed by a custom CMS built by us in the old days, which hosted in a separate domain. After the issue stricken, we did couple of things,

- Changed DB password.
- Cms access denied.
- Enabled all the Diagnostics modules in the **[Everleap][2]** control panel. (HTTP Logging, Detailed Error Logging, Failed Request Tracing).


So we concluded that the website is being compromised. Further actions,

We then blocked the CMS access to identify whether the CMS or the site is being compromised. The next day evening, the same thing happened. So we concluded that the website is being compromised. Next steps,


- Changed DB password again.
- Code review

### Code review
Code review went okay, the application was built on top of Web forms. And there were 3 areas where user input is taken,

- Search
- Email Subscription
- Contact form

In which Contact form input didn't touch database since only email alert configured. So for the other two server-side input validation tightened. We also tried some ethical hacking attempts but all of them got blocked by **[Sucuri][3]**.

Checked the site security status in the [Asafaweb](https://asafaweb.com/) and the result screenshot is given below, 
![asfaweb-result][4]
Recommended changes for **Request Validation** were done and all the updates were deployed but on the same evening, it happened again!!

### Next steps
Since the project was kind of outdated, the manager asked us to estimate for the project migration into Umbraco since we couldn't find anything suspicious in the coding, YET. So I delegated the estimation task to my colleague and started to have a look in the HTTP log downloaded from the **[Everleap][2]**. 

### Turning point
The log file contained 10k+ lines, my first search was for "POST" requests. While navigating through search results I noticed a get request with parameter contains web.config. The log for the same was,

> 2017-09-24 16:44:10 1990-15390 GET /PDF/DownLoad.aspx
> fileName=../web.config&X-ARR-LOG-ID=1c936dd2-8679-42ad-b77c-da8a938059a4
> 80 xx-xxxxx-x-x-x-x-x--xx--x-x

The above request was handled by the PDF download handler module, its intended function was to provide the requested PDF file. 

So I checked the request handler and here is the code,

<pre style="font-family:Fantasque Sans Mono;font-size:13;color:gainsboro;background:#1e1e1e;"><span style="color:#569cd6;">protected</span>&nbsp;<span style="color:#569cd6;">void</span>&nbsp;<span style="color:cyan;">Page_Load</span>(<span style="color:#569cd6;">object</span>&nbsp;sender,&nbsp;<span style="color:lightblue;">EventArgs</span>&nbsp;e)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#569cd6;">string</span>&nbsp;fileName&nbsp;<span style="color:#b4b4b4;">=</span>&nbsp;<span style="color:violet;">Request</span><span style="color:#b4b4b4;">.</span><span style="color:violet;">QueryString</span>[<span style="color:#d69d85;">&quot;fileName&quot;</span>]<span style="color:#b4b4b4;">.</span><span style="color:cyan;">ToString</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:violet;">Response</span><span style="color:#b4b4b4;">.</span><span style="color:violet;">ContentType</span>&nbsp;<span style="color:#b4b4b4;">=</span>&nbsp;<span style="color:#d69d85;">&quot;application/octet-stream&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:violet;">Response</span><span style="color:#b4b4b4;">.</span><span style="color:cyan;">AddHeader</span>(<span style="color:#d69d85;">&quot;Content-Disposition&quot;</span>,&nbsp;<span style="color:#d69d85;">&quot;attachment;filename=&quot;</span>&nbsp;<span style="color:#b4b4b4;">+</span>&nbsp;fileName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:violet;">Response</span><span style="color:#b4b4b4;">.</span><span style="color:cyan;">TransmitFile</span>(&nbsp;fileName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:violet;">Response</span><span style="color:#b4b4b4;">.</span><span style="color:cyan;">End</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</pre>

The above code is doing its intended function very well but it can do unintended function as well **if you know what I mean**. To prove my point, I tried to invoke the same handler with the suspicious input and boom, here comes the requested file.

### Final steps
Removed the request handler, changed DB password again. After the bug patched, I checked the HTTP log again in the next day and found out that there is an exact same request but this time instead of 200, a 404 response.  And now weeks passed, website working fine, so far so good.


  [1]: https://in.godaddy.com/
  [2]: http://www.everleap.com/a/oybnbo
  [3]: http://sucuri.net
  [4]: /media/1045/asafaweb-result.png
## Image
### Post Image
![Post Image](neonbrand-356967.jpg) 
### Post Header Image
![Post Header Image](ruben-bagues-135824.jpg)

## Meta Tags
### Social Description
In this post, I am sharing one of my security bug hunting experience in an older ASP.NET web form project.
