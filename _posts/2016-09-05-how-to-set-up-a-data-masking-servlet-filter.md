---
id: 637
title: How to Set Up a Data Masking Servlet Filter
date: 2016-09-05T18:00:41+00:00
guid: https://www.peoplesoftmods.com/?p=637
permalink: /servlet-filters/how-to-set-up-a-data-masking-servlet-filter/
categories:
  - Data Masking
  - Servlet Filters
---
I previously demonstrated how servlet filters can be used to view and modify HTTP requests that the client sends to the web server.  This post will demonstrate how servlet filters can view and modify the HTTP responses that the web server sends back to the client.  The servlet filter that will be used in this demonstration is one to mask social security numbers (SSNs) that appear in the response messages.  Using servlet filters to perform sensitive data masking does not change the actual value of the data in the database, but it still protects the true values from being exposed through the PIA.

<!--more-->

[**CLICK HERE**](https://www.peoplesoftmods.com/Development/PSM_DATA_MASKER_POC.zip) to download the project.

Extract the zip and you should see the following files:

[<img class="alignnone size-full wp-image-640" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/1extract.png" alt="extract" width="549" height="245" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/1extract.png 549w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/1extract-300x134.png 300w" sizes="(max-width: 549px) 100vw, 549px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/1extract.png)

Take the folder named “custom” and place it in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF\classes\com\peoplesoft\pt</pre>

[<img class="alignnone size-full wp-image-641" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/2customfolder.png" alt="customfolder" width="616" height="342" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/2customfolder.png 616w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/2customfolder-300x167.png 300w" sizes="(max-width: 616px) 100vw, 616px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/2customfolder.png)

Take the folder named “DataMaskingFilter” and place it in the following directory:

<pre> %PS_HOME%\sdk</pre>

[<img class="alignnone size-full wp-image-642" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3dmffolder.png" alt="dmffolder" width="647" height="352" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3dmffolder.png 647w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3dmffolder-300x163.png 300w" sizes="(max-width: 647px) 100vw, 647px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3dmffolder.png)

Navigate to and open up the web.xml file.  The web.xml file is located in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF</pre>

[<img class="alignnone size-full wp-image-643" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3webxml.png" alt="webxml" width="675" height="350" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3webxml.png 675w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3webxml-300x156.png 300w" sizes="(max-width: 675px) 100vw, 675px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/3webxml.png)

Copy the text from the downloaded file named “webxml.txt” and paste the text into your web.xml file.  Paste in the text so that it is the first <filter> to show up in the web.xml file.  For my environment, I pasted the filter before the delivered “psfilter”.

[<img class="alignnone size-full wp-image-644" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/4dmfwebxml.png" alt="dmfwebxml" width="680" height="353" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/4dmfwebxml.png 680w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/4dmfwebxml-300x156.png 300w" sizes="(max-width: 680px) 100vw, 680px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/4dmfwebxml.png)

Go to the web profile configuration page in the PIA and select the web profile that you are applying this servlet filter to.  You will need to uncheck the checkbox for the &#8220;Compress Responses&#8221; option.

[<img class="alignnone size-full wp-image-645" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/5webprofile.png" alt="webprofile" width="806" height="258" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/5webprofile.png 806w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/5webprofile-300x96.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/5webprofile-768x246.png 768w" sizes="(max-width: 806px) 100vw, 806px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/5webprofile.png)

If you would like to keep the compression enabled on the web profile, then you can implement [my response compression servlet filter here](https://www.peoplesoftmods.com/servlet-filters/response-compression-with-custom-servlet-filters/).

**You will need to bounce the web server at this point.**

After bouncing the web server, login to the PIA and navigate to a page that would normally expose an SSN value.  In my Campus Solutions environment, I navigated to the Relationships page and saw the following:

[<img class="alignnone size-full wp-image-646" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships.png" alt="Relationships" width="1214" height="468" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships.png 1214w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships-300x116.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships-768x296.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships-1024x395.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships-986x380.png 986w" sizes="(max-width: 1214px) 100vw, 1214px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6Relationships.png)

Similarly, I got the following when I went to the Add/Update a Person Page:

[<img class="alignnone size-full wp-image-647" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson.png" alt="AddUpdatePerson" width="1075" height="505" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson.png 1075w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson-300x141.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson-768x361.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson-1024x481.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson-809x380.png 809w" sizes="(max-width: 1075px) 100vw, 1075px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/6AddUpdatePerson.png)

I also tested the html output of a PS Query that returned SSNs.

[<img class="alignnone size-full wp-image-648" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/7selectquery.png" alt="selectquery" width="1024" height="343" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/7selectquery.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/7selectquery-300x100.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/7selectquery-768x257.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/7selectquery.png)

The output of the query only showed masked SSNs.

[<img class="alignnone size-full wp-image-649" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery.png" alt="viewquery" width="1580" height="368" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery.png 1580w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery-300x70.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery-768x179.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery-1024x239.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery-1180x275.png 1180w" sizes="(max-width: 1580px) 100vw, 1580px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/09/8viewquery.png)

The masking capabilities of this servlet filter can add an additional layer of security without having to modify the delivered PeopleSoft application.  This servlet filter makes use of Java regular expressions to find and mask the SSN data.  The use of regular expressions can allow for easily finding, replacing, and/or inserting certain information into the HTTP responses.  For example, one interesting use case of using regular expressions with servlet filters is for injecting JavaScript into the header tag of each html response that is sent to the client&#8217;s browser.

The demonstrated functionality of this servlet filter is powerful, but it is not very robust.  A somewhat easy way to improve the robustness would be to allow for conditionally masking data based on the user&#8217;s location and/or if the user has successfully performed 2FA for the session.  As demonstrated, the functionality is either all or nothing based on whether the filter is enabled in the web.xml file.  This provides for a rather poor user experience and can be problematic under certain circumstances.  The servlet filter can be enhanced to reference custom session variables or custom cookie values to determine if a mask is needed.

An improvement in a different direction would be to provide a separate complementary functionality. The complementary functionality would provide the ability for the masked data to be interactive with custom injected client side code that can enable the following scenarios:

  * Click To View &#8211; A user will click the masked value on a page to view the unmasked version. Under the hood, the click event will make a request to an iscript that will return the unmasked value to replace the masked value on the page.

  * Click To Challenge To View &#8211; In this scenario, A user will click the masked value on a page, but will be challenged (2FA) to view the unmasked version.  The click event will call an iscript that will return a modal window to be displayed for the user. The modal window will challenge the user for 2FA. Inputting a correct TOTP into the 2FA modal window will invoke an iscript to return the unmasked value to replace the masked value on the page.

Each of the &#8220;Click To&#8221; scenarios shall use AJAX to not lose the page&#8217;s originally loaded state. Also, the iscripts that get called on the click events will log important details of the transaction such as the emplid requesting the data, the menu/component/page names, and time.

The brilliant solution to enable the &#8220;Click To&#8221; functionality to expose the unmasked data/log the event was inspired by <a href="http://www.greyheller.com/products/erp-firewall/" target="_blank">Grey Heller&#8217;s ERP firewall solution</a>.  I recently had the pleasure of chatting with one of the Grey Heller representatives.  The representative had explained the importance of the ability to capture and log the details of the sensitive data exposure event. Enforcing the user to click a button to view a sensitive piece of data provides for an avenue to log the details of the specific transaction.  If you do not enforce an event like a button click to view a sensitive piece of data , then you lose the desired ability to log at a granular level.

In the future, I plan to expand on the functionality demonstrated in this post by adding the additional functionality described above.