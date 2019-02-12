---
id: 393
title: How to Set Up a Redirection Servlet Filter in PeopleSoft
date: 2016-07-20T13:18:52+00:00
guid: https://www.peoplesoftmods.com/?p=393
permalink: /2fa/how-to-setup-a-redirection-servlet-filter-in-peoplesoft/
categories:
  - Servlet Filters
  - Two-Factor Authentication
---
I would like to provide a tutorial to show how redirection capabilities can be achieved with servlet filters.  There are many use cases for why you would need to redirect a user’s browser with a servlet filter.  For example, there might be some resources in your PeopleSoft application that you want to only allow access to under certain conditions. When these conditions are not met, then you might want to redirect users to a different page.  Some example conditions can be things like: the user must be on an internal IP address, the user must present a valid secondary authentication token, or the time of the day must be between 8:00 am – 5:00 pm. Anything that is presented in an HTTP request can be analyzed to see if it meets a certain criteria.

<!--more-->

I [previously demonstrated](https://www.peoplesoftmods.com/servlet-filters/servlet-filters-in-peoplesoft/) the redirection capabilities of servlet filters with the enforcement of two-factor authentication (2FA). In that demonstration, users that did not present a valid 2FA token when they attempted to access certain resources would be redirected to the 2FA page.

I am going to provide a proof-of-concept example of how to do the redirection portion.  What this example will be doing is checking if a user has a valid token when they attempt to access a resource in the PeopleSoft application where the URL contains “/psc/”.  If the user does not present a valid token, then the user will be redirected to a static html page where they can get a valid token.  Once the user obtains the token from the html page, the user will be redirected back to the resource that they were originally trying to access.  If the user loses or doesn’t present this token when they try to access any “/psc/” resources, then they will be redirected to get a new token.  Note: A &#8220;valid&#8221; token in this example is merely a correctly named token.  The actual value of the token is disregarded in this example for simplicity.

* * *

[<span style="text-decoration: underline;"><strong>CLICK HERE</strong></span>](/assets/downloads/PSM_SERVLET_FILTER_POC.zip) to download the project.

Extract the zip and you should see the following files:

[<img class="alignnone size-full wp-image-394" src="/assets/images/2016/07/Files.png" alt="Files" width="182" height="100" />](/assets/images/2016/07/Files.png)

Take the folder named “custom” and place it in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF\classes\com\peoplesoft\pt</pre>

[<img class="alignnone size-full wp-image-395" src="/assets/images/2016/07/ClassFile.png" alt="ClassFile" width="643" height="274" srcset="/assets/images/2016/07/ClassFile.png 643w, /assets/images/2016/07/ClassFile-300x128.png 300w" sizes="(max-width: 643px) 100vw, 643px" />](/assets/images/2016/07/ClassFile.png)

Take the folder named “RedirectFilter” and place it in the following directory:

<pre>%PS_HOME%\sdk</pre>

[<img class="alignnone size-full wp-image-396" src="/assets/images/2016/07/RedirectFilter.png" alt="RedirectFilter" width="759" height="395" srcset="/assets/images/2016/07/RedirectFilter.png 759w, /assets/images/2016/07/RedirectFilter-300x156.png 300w, /assets/images/2016/07/RedirectFilter-730x380.png 730w" sizes="(max-width: 759px) 100vw, 759px" />](/assets/images/2016/07/RedirectFilter.png)

Take the file named “SetToken.html” and place it in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war</pre>

[<img class="alignnone size-full wp-image-397" src="/assets/images/2016/07/SetToken.png" alt="SetToken" width="729" height="265" srcset="/assets/images/2016/07/SetToken.png 729w, /assets/images/2016/07/SetToken-300x109.png 300w" sizes="(max-width: 729px) 100vw, 729px" />](/assets/images/2016/07/SetToken.png)

Copy the text from the file named “Paste this into webxml” and paste the text into your web.xml file.  The web.xml file is located in the following directory:

<pre>%PS_CFG_HOME%\webserv\*your_domain_name*\applications\peoplesoft\PORTAL.war\WEB-INF</pre>

Depending on how modified your web.xml file is, you might want to paste the text in a different part than me, but I pasted it in as the first filter (just before the “psfilter”).

[<img class="alignnone size-full wp-image-398" src="/assets/images/2016/07/webxml.png" alt="webxml" width="613" height="431" srcset="/assets/images/2016/07/webxml.png 613w, /assets/images/2016/07/webxml-300x211.png 300w, /assets/images/2016/07/webxml-540x380.png 540w" sizes="(max-width: 613px) 100vw, 613px" />](/assets/images/2016/07/webxml.png)

**You will need to bounce the web server after you perform all of these steps.**

After bouncing the web server, navigate to the SetToken.html file in your web browser.  You should be prompted to input a token name.  You need to input the same name that  was specified in the param-value of the “TokenName” parameter in the web.xml.  As you see, I set the token name to “MY_TOKEN” in the xml that I provided.

[<img class="alignnone size-full wp-image-399" src="/assets/images/2016/07/TokenName.png" alt="TokenName" width="383" height="254" srcset="/assets/images/2016/07/TokenName.png 383w, /assets/images/2016/07/TokenName-300x199.png 300w" sizes="(max-width: 383px) 100vw, 383px" />](/assets/images/2016/07/TokenName.png)

So I will input the value “MY_TOKEN” into the prompt.

[<img class="alignnone size-full wp-image-400" src="/assets/images/2016/07/TokenPrompt.png" alt="TokenPrompt" width="758" height="316" srcset="/assets/images/2016/07/TokenPrompt.png 758w, /assets/images/2016/07/TokenPrompt-300x125.png 300w" sizes="(max-width: 758px) 100vw, 758px" />](/assets/images/2016/07/TokenPrompt.png)

You will obtain the token after clicking OK on the prompt. You can check the cookies in your browser to see this.

[<img class="alignnone size-full wp-image-401" src="/assets/images/2016/07/Token.png" alt="Token" width="693" height="187" srcset="/assets/images/2016/07/Token.png 693w, /assets/images/2016/07/Token-300x81.png 300w" sizes="(max-width: 693px) 100vw, 693px" />](/assets/images/2016/07/Token.png)

You should be able to login and use your PeopleSoft application like you normally would.  If you delete the token from your browser, then the next “/psc/” resource that you try to load will redirect you to the SetToken page so that you can get the needed token.  The SetToken page should then redirect you back to the resource that you were trying to access.

* * *

This is essentially how I enforce 2FA in my PeopleSoft application.  The SetToken page is like the 2FA page where the user has to input a time sensitive code.  The “MY\_TOKEN” cookie in the browser is the 2FA token that gets issued to users that successfully perform 2FA.  So the “MY\_TOKEN” in the user’s browser signifies that the user has done 2FA for the session.  This proof-of-concept can also be applied to a slightly different context of single sign on or using a third party to authenticate PeopleSoft users.  In this context, the SetToken page will be the third-party login page that will authenticate the user.  The user gets the authentication token and takes it over to PeopleSoft and this servlet filter can verify that the token is legit and allow access accordingly.  An additional sign on PeopleCode function would also most likely be needed to achieve this sort of SSO functionality.

One more thing that I would like to touch on is how you can enforce the redirection for as many or few resource in the PeopleSoft application that you want. You can set the url-pattern in the web.xml to “/*”.  This will fire the filter for every request.  If you are interested in enforcing the redirection at specific PeopleSoft components, then you can check (In the Java code) if the resource that the user is attempting to access matches the MENU.COMPONENT of the component that you would like to protect.  You can also achieve this same redirection functionality by [mapping application class PeopleCode](https://www.peoplesoftmods.com/2fa/using-the-event-mapping-framework-to-enforce-two-factor-authentication/) to components that you would like to protect with the use of the event mapping framework.

If you would like to see how to implement another PeopleSoft-related servlet filter, then check out <a href="https://github.com/psadmin-io/ps-eat-cookies" target="_blank">Kyle Benson’s ps-eat-cookies filter</a>.  This is a great example of how to modify the response that your web server sends to the client’s machine.  If you are interested in learning more about how servlet filters work in PeopleSoft, then I highly recommend listening to this <a href="http://psadmin.io/2016/04/29/26-weblogic-filters/" target="_blank">podcast episode about WebLogic filters</a>.