---
id: 509
title: How to Set Up a Request Logging Servlet Filter
date: 2016-08-14T14:44:48+00:00
guid: https://www.peoplesoftmods.com/?p=509
permalink: /servlet-filters/how-to-set-up-a-request-logging-servlet-filter/
categories:
  - Logging
  - Servlet Filters
---
I developed a servlet filter that is capable of logging the request data that a client sends to the PeopleSoft servlets.  I did this project to better familiarize myself with Java as well as to get more comfortable developing at the web-tier.  This servlet filter can be useful for keeping track of what users are doing in your PeopleSoft applications.  I will admit that the code that I am releasing here is pretty raw and should be used with caution.  I am mostly putting this out here for documentation purposes.

<!--more-->

[**CLICK HERE**](https://www.peoplesoftmods.com/Development/PSM_REQUEST_LOGGER_POC.zip) to download the project.

Extract the zip and you should see the following files:

[<img class="alignnone wp-image-510 size-full" src="/assets/images/2016/08/files.png" alt="files" width="141" height="68" />](/assets/images/2016/08/files.png)

Take the folder named “custom” and place it in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF\classes\com\peoplesoft\pt</pre>

[<img class="alignnone wp-image-511 size-full" src="/assets/images/2016/08/custom_folder.png" alt="custom_folder" width="590" height="270" srcset="/assets/images/2016/08/custom_folder.png 590w, /assets/images/2016/08/custom_folder-300x137.png 300w" sizes="(max-width: 590px) 100vw, 590px" />](/assets/images/2016/08/custom_folder.png)

Take the folder named “RequestLoggerFilter” and place it in the following directory:

<pre> %PS_HOME%\sdk</pre>

[<img class="alignnone size-full wp-image-514" src="/assets/images/2016/08/sdk_folder.png" alt="sdk_folder" width="590" height="393" srcset="/assets/images/2016/08/sdk_folder.png 590w, /assets/images/2016/08/sdk_folder-300x200.png 300w, /assets/images/2016/08/sdk_folder-570x380.png 570w" sizes="(max-width: 590px) 100vw, 590px" />](/assets/images/2016/08/sdk_folder.png)

Navigate to and open up the web.xml file.  The web.xml file is located in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF</pre>

[<img class="alignnone size-full wp-image-515" src="/assets/images/2016/08/webxml_file.png" alt="webxml_file" width="623" height="292" srcset="/assets/images/2016/08/webxml_file.png 623w, /assets/images/2016/08/webxml_file-300x141.png 300w" sizes="(max-width: 623px) 100vw, 623px" />](/assets/images/2016/08/webxml_file.png)

Copy the text from the downloaded file named “webxml.txt” and paste the text into your web.xml file.  Paste in the text so that it is the first <filter> to show up in the web.xml file.  For my environment, I pasted the filter before the delivered “psfilter”.  After saving the web.xml file, **you will need to bounce the web server.**

[<img class="alignnone size-full wp-image-516" src="/assets/images/2016/08/webxml_text.png" alt="webxml_text" width="724" height="706" srcset="/assets/images/2016/08/webxml_text.png 724w, /assets/images/2016/08/webxml_text-300x293.png 300w, /assets/images/2016/08/webxml_text-390x380.png 390w" sizes="(max-width: 724px) 100vw, 724px" />](/assets/images/2016/08/webxml_text.png)

After bouncing the web server, login to the PIA and navigate around to some arbitrary components.  I logged in as the PS user and navigated to the “Change my Password” page. Now open up the following directory on the webserver:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\</pre>

You should see a csv file that is prefixed with “PSM\_REQLOG\_”

[<img class="alignnone size-full wp-image-517" src="/assets/images/2016/08/output_file.png" alt="output_file" width="633" height="554" srcset="/assets/images/2016/08/output_file.png 633w, /assets/images/2016/08/output_file-300x263.png 300w, /assets/images/2016/08/output_file-434x380.png 434w" sizes="(max-width: 633px) 100vw, 633px" />](/assets/images/2016/08/output_file.png)

Open this file up and it will contain request logs.  Since I logged in as the PS account and navigated to the “Change My password’ page, my log file contains the following:

[<img class="alignnone size-full wp-image-518" src="/assets/images/2016/08/output.png" alt="output" width="736" height="83" srcset="/assets/images/2016/08/output.png 736w, /assets/images/2016/08/output-300x34.png 300w" sizes="(max-width: 736px) 100vw, 736px" />](/assets/images/2016/08/output.png)

There were two lines that showed up for this transaction.  The first one was for the call to the &#8220;psp&#8221; servlet and the second one was for the call to the &#8220;psc&#8221; servlet.  If you are interested in logging the requests for only a single servlet, then scroll to the &#8220;Configuring the URL Pattern&#8221; section.  The third column of the output file is the contents that are stored in the USERID session variable that is defined on the web server.  This value contains the PeopleSoft ID and IP address that made the request.  Scroll down to the &#8220;Attributes Parameter&#8221; section to get more information on logging these types of values.

_Below is an explanation of the different input parameters for this servlet filter.  These parameters can be used to specify the type of requests and information that you are interested in logging._

&nbsp;

**ContentTypes Parameter:**

This parameter is used to specify which requested content types that you want to be logged. You can specify a single content type, multiple content types (csv format), or a * to log all content types. A typical PeopleSoft URL will have the following structure:

http://server/servlet_name/sitename/portalname/nodename/**content_type**/content\_id?content\_parm

The content_type section of the URL is usually denoted by a single character. This single character representation of the content type shall be specified for the ContentTypes parameter. So if you want to log component content types, then the ContentTypes parameter will look like this:

<pre>&lt;init-param&gt;

&lt;param-name&gt;ContentTypes&lt;/param-name&gt;

&lt;param-value&gt;c&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

Or if you want to log component, script, and query content types, then the ContentTypes parameter will look like this:

<pre>&lt;init-param&gt;

&lt;param-name&gt;ContentTypes&lt;/param-name&gt;

&lt;param-value&gt;c,s,q&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

Or you can log all content types with the following setting:

<pre>&lt;Init-param&gt;

&lt;param-name&gt;ContentTypes&lt;/param-name&gt;

&lt;param-value&gt;*&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

Search “URL Format for PeopleSoft Pure Internet Architecture Content Types” in PeopleBooks to get a listing of the different content types.

&nbsp;

**LogQueryStr Parameter:**

This parameter is used to specify whether or not you want to log the URL query parameters that get sent for some requests. A typical PeopleSoft URL will have the following structure:

http://server/servlet\_name/sitename/portalname/nodename/content\_type/content_id?**content_parm**

The content_parm section of the URL refers to the query parameters. If you would like for this information to be logged, then specify a 1 for the LogQueryStr parameter as follows:

<pre>&lt;Init-param&gt;

&lt;param-name&gt;LogQueryStr&lt;/param-name&gt;

&lt;param-value&gt;1&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

Or if you do not want this information to be logged, then you can specify a 0 for the LogQueryStr parameter as follows:

<pre>&lt;Init-param&gt;

&lt;param-name&gt;LogQueryStr&lt;/param-name&gt;

&lt;param-value&gt;0&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

&nbsp;

**LogBody Parameter:**

This parameter is used to specify whether or not you want to log the request message body that gets sent for some requests. Information gets sent in the request body for actions such as logging in, saving a component, or clicking search on a search page. You can capture this information by specifying a 1 for the LogBody parameter as follows:

<pre>&lt;Init-param&gt;

&lt;param-name&gt;LogBody&lt;/param-name&gt;

&lt;param-value&gt;1&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

If you do not want this information to be logged, then you can specify a 0 for the LogBody parameter as follows:

<pre>&lt;Init-param&gt;

&lt;param-name&gt;LogBody&lt;/param-name&gt;

&lt;param-value&gt;0&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

**Note:** This functionality should be used with caution. Any request body data that is captured will be logged as plain text. This means that <span style="color: #ff0000;"><strong>any sensitive data that gets sent with a request will</strong> <strong>be stored as plain text in the log file</strong></span>.

&nbsp;

**Attributes Parameter:**

This parameter is used to capture session variable information that is stored on the web server during a given request. You can specify the session variable names (csv format) that you would like to capture the value for. The names and existence of the session variables can potentially change with each PeopleTools version release. So if you are unsure of the specific session variable names that exist for your current PeopleTools release, then you can specify a * to dump all of the session name-value pairs for requests.

<pre>&lt;Init-param&gt;

&lt;param-name&gt;Attributes&lt;/param-name&gt;

&lt;param-value&gt;*&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

After reviewing the captured name-value pairs, you can then specify the names of the session variables that you would like to capture. In the 8.55.06 PeopleTools release that I tested this servlet filter on, there exist session variables name USERID and ROLES (along with other variables). To log the values of these session variables for each request, then you can specify the Attributes parameter as follows:

<pre>&lt;Init-param&gt;

&lt;param-name&gt;Attributes&lt;/param-name&gt;

&lt;param-value&gt;USERID,ROLES&lt;/param-value&gt;

&lt;/init-param&gt;</pre>

&nbsp;

**FileName Parameter:**

This parameter is used to specify the output filename that the logs will get written to. The actual output log filename will be the value that is given for this parameter with the current date (MM\_DD\_YY format) appended to it. The generated output logs will be saved in csv format.

&nbsp;

**Configuring the URL Pattern:**

If you only want the requests to be logged when a particular servlet is called, then you can specify this with the filter-mapping configuration. A typical PeopleSoft URL will have the following structure:

http://server/**servlet_name**/sitename/portalname/nodename/content\_type/content\_id?content_parm

You can filter what is logged by specifying the servlet_name section of the URL in the filter-mapping configuration. If you want only the requests for the “psc” servlet to be logged, then you can configure the filter-mapping like this:

<pre>&lt;filter-mapping&gt;

&lt;filter-name&gt;RequestLoggerFilter&lt;/filter-name&gt;

&lt;url-pattern&gt;/psc/*&lt;/url-pattern&gt;

&lt;/filter-mapping&gt;</pre>

Or you can specify /* to log the requests made to all of the different servlets:

<pre>&lt;filter-mapping&gt;

&lt;filter-name&gt;RequestLoggerFilter&lt;/filter-name&gt;

&lt;url-pattern&gt;/*&lt;/url-pattern&gt;

&lt;/filter-mapping&gt;</pre>

Refer to PeopleBooks for a listing of the available delivered servlets.

* * *

Most of the functionality that this servlet filter provides can be achieved via <a href="http://docs.oracle.com/cd/E13222_01/wls/docs90/config_wls/web_server.html#1059425" target="_blank">configuration on the WebLogic server</a>.  However,  coding and implementing this functionality myself was a great learning experience.  I believe that this example servlet filter can be used as a base to create more useful servlet filters.  This request logging servlet filter and this [redirection servlet filter](https://www.peoplesoftmods.com/2fa/how-to-setup-a-redirection-servlet-filter-in-peoplesoft/) serve as examples of viewing and modifying request data that the client sends to the web server.  In the future, I hope to provide an example of a data masking servlet filter that works on the response data that the web server generates for the client.