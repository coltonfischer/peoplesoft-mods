---
id: 805
title: Response Compression with Custom Servlet Filters
date: 2017-01-08T14:02:22+00:00
guid: https://www.peoplesoftmods.com/?p=805
permalink: /servlet-filters/response-compression-with-custom-servlet-filters/
categories:
  - Servlet Filters
---
One major drawback of the response-manipulating servlet filters that I have showcased on this site is that the delivered compression functionality needed to be disabled for them to work.  This involved unchecking the “Compress Responses” checkbox on the web profile configuration page in the PIA.  Disabling this functionality causes a performance impact because of the resulting large response messages that get sent to the client.  I recently got a [comment from Jonathan Rehm](https://www.peoplesoftmods.com/servlet-filters/how-to-set-up-a-data-masking-servlet-filter/#comment-5021) explaining how he has achieved GZIP compression within a custom servlet filter.  I decided to integrate the code he shared into a standalone response compression servlet filter.  This servlet filter can co-exist with any other custom filters that you may have deployed.  I will demonstrate how to deploy this custom response compression servlet filter in this post.

<!--more-->

[**CLICK HERE**](https://www.peoplesoftmods.com/Development/PSM_COMPRESSION_FILTER_POC.zip) to download the project.

Extract the zip and you should see the following files:

[<img class="alignnone size-full wp-image-808" src="/assets/images/2017/01/1.png" alt="Compression Filter Files" width="549" height="239" srcset="/assets/images/2017/01/1.png 549w, /assets/images/2017/01/1-300x131.png 300w" sizes="(max-width: 549px) 100vw, 549px" />](/assets/images/2017/01/1.png)

Take the folder named “custom” and place it in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF\classes\com\peoplesoft\pt</pre>

[<img class="alignnone size-full wp-image-809" src="/assets/images/2017/01/2.png" alt="Class Files" width="845" height="367" srcset="/assets/images/2017/01/2.png 845w, /assets/images/2017/01/2-300x130.png 300w, /assets/images/2017/01/2-768x334.png 768w" sizes="(max-width: 845px) 100vw, 845px" />](/assets/images/2017/01/2.png)

Navigate to and open up the web.xml file.  The web.xml file is located in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF</pre>

[<img class="alignnone size-full wp-image-810" src="/assets/images/2017/01/3.png" alt="web.xml File" width="678" height="363" srcset="/assets/images/2017/01/3.png 678w, /assets/images/2017/01/3-300x161.png 300w" sizes="(max-width: 678px) 100vw, 678px" />](/assets/images/2017/01/3.png)

Copy the text from the downloaded file named “webxml.txt” and paste the text into your web.xml file.  Paste in the text so that it is the first <filter> to show up in the web.xml file. This compression filter must be placed before any of your custom response-manipulating filters.

[<img class="alignnone size-full wp-image-811" src="/assets/images/2017/01/4.png" alt="Custom Filters" width="900" height="929" srcset="/assets/images/2017/01/4.png 900w, /assets/images/2017/01/4-291x300.png 291w, /assets/images/2017/01/4-768x793.png 768w, /assets/images/2017/01/4-368x380.png 368w" sizes="(max-width: 900px) 100vw, 900px" />](/assets/images/2017/01/4.png)

**You will need to bounce the web server at this point.**

After bouncing the web server, all of the HTTP responses that are being sent to the client should be getting compressed via this custom filter. This is true even for responses that may be getting manipulated by some other custom filters such as a [data masking filter](https://www.peoplesoftmods.com/servlet-filters/how-to-set-up-a-data-masking-servlet-filter/) or a [global script injection filter](https://www.peoplesoftmods.com/servlet-filters/global-script-and-style-injection/).  Here is an example of a response message that got sent to a client with the compression filter disabled:

[<img class="alignnone size-full wp-image-812" src="/assets/images/2017/01/6.png" alt="Compression Disabled" width="914" height="838" srcset="/assets/images/2017/01/6.png 914w, /assets/images/2017/01/6-300x275.png 300w, /assets/images/2017/01/6-768x704.png 768w, /assets/images/2017/01/6-414x380.png 414w" sizes="(max-width: 914px) 100vw, 914px" />](/assets/images/2017/01/6.png)

Notice how large the response body of the message is.  Now here is the same request with this custom servlet filter enabled:

[<img class="alignnone size-full wp-image-813" src="/assets/images/2017/01/7.png" alt="Compression Enabled" width="919" height="871" srcset="/assets/images/2017/01/7.png 919w, /assets/images/2017/01/7-300x284.png 300w, /assets/images/2017/01/7-768x728.png 768w, /assets/images/2017/01/7-401x380.png 401w" sizes="(max-width: 919px) 100vw, 919px" />](/assets/images/2017/01/7.png)

Notice how the response body of this message is significantly smaller (by 86.2%) than the non-compressed version.

With this custom compression filter enabled, it does not matter if the web profile is configured to compress responses.  This custom compression filter will compress responses regardless.  So the following box on the web profile configuration page in the PIA can be checked or unchecked.

[<img class="alignnone size-full wp-image-814" src="/assets/images/2017/01/5.png" alt="Web Profile" width="787" height="284" srcset="/assets/images/2017/01/5.png 787w, /assets/images/2017/01/5-300x108.png 300w, /assets/images/2017/01/5-768x277.png 768w" sizes="(max-width: 787px) 100vw, 787px" />](/assets/images/2017/01/5.png)

A nice enhancement to this solution would be to read in the web server properties file that contains the &#8220;Compress Responses&#8221; boolean and to disable/enable the custom compression filter accordingly.  As for now this &#8220;Compress Responses&#8221; value is disregarded.

So you may be wondering: How does this custom filter override the delivered compression functionality?  The answer is by using the HttpServletRequestWrapper class to wrap the original request message that was sent from the client. The original request message will contain the Accept-Encoding header that specifies the type of encoding that the browser is capable of receiving. Since the delivered compression functionality occurs within the actual servlet, the Accept-Encoding header is what the servlet will look at to determine if compression should be performed or not.  What this custom wrapped request does is it spoofs the Accept-Encoding header to null. This means that when the servlet receives the wrapped request, it does not perform compression because the Accept-Encoding header is null. This results in the servlet to send a non-compressed response message back down the filter chain.  That means when a custom response-manipulating servlet filter receives the response message, it can easily modify the response body of the message since it is not compressed. With this custom response compression filter being first in the order specified in the web.xml file,  it will be the last filter to touch the response message.  What this filter does is it references the Accept-Encoding header of the original request message and it compresses the response accordingly.

It is worth noting that this filter is only capable of encoding responses with GZIP.  This means that if a client does not specify GZIP as an acceptable encoding scheme in the Accept-Encoding header, then no compression will be performed.