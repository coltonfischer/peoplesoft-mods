---
id: 147
title: Setting Up Two-Factor Authentication in PeopleSoft (Part 2)
date: 2016-06-06T13:35:26+00:00
guid: https://www.peoplesoftmods.com/?p=147
permalink: /2fa/setting-up-two-factor-authentication-in-peoplesoft-part-2/
categories:
  - Two-Factor Authentication
---
In this part of the tutorial, I have expanded on what was done in the <a href="https://www.peoplesoftmods.com/2fa/how-to-set-up-two-factor-authentication-in-peoplesoft/" target="_blank">first part</a> by adding additional logic to the Signon PeopleCode and adding more functionality to the 2FA page.  I went ahead and created an app designer project that contains all of the objects and code that serves as a proof-of-concept of how the 2FA process works as a whole.  The only functionality that this project does not contain is the ability to send SMS messages. You should be able to plug and play this project into your environment.

<!--more-->

I tried my best to explain what all is going on in the comments in the code, but here some important features that I would like to touch on:

  * The 2FA process is enforced only on users that have the Peoplesoft Administrator role that do not possess a valid 2FA cookie in their browser. The code can easily be changed to challenge users for 2FA with more or different roles.
  * The cookie name that gets written to the user’s browser will be in the form of PSM\_2FA\_TOKEN\_%USERID\_%DBNAME

<p style="padding-left: 60px;">
  The %DBNAME part of the cookie’s name allows the user to poses tokens for multiple environments if they wish. This assumes that this 2FA process is implemented on multiple environments.
</p>

<p style="padding-left: 60px;">
  The %USERID part of the cookie’s name is used to accommodate for multiple users that might be sharing the same browser.
</p>

  * The cookie that is written to the user’s browser has a value that contains a hash of the user’s %USERID, %IP_ADDRESS, and %DBNAME.

<p style="padding-left: 60px;">
  This is what will make the cookie only valid for a specific user logging into a specific environment from a specific location.  So for example, assume a user gets a 2FA token on their laptop at work to get into the CSPRD environment. The user goes home with their laptop and tries to use the previously obtained token to bypass 2FA from their home network when logging into the CSPRD environment. The user will be challenged for 2FA again since the token that they possess is only valid for their IP address at work.
</p>

Note: The details that I explained above are only really relevant if you use the &#8220;Remember This Device&#8221; functionality, otherwise the token will be deleted at the end of the session.  The &#8220;Remember This Device&#8221; functionality gives the user the option to not have to do 2FA on every login.  This functionality was provided for user experience purposes, but if you are looking to have a more secure solution, then do not provide this functionality to the users.

Below are the steps to get the project up and running.

<span style="color: #ff0000;">UPDATE</span>: It was discovered that this project has some compatibility issues for environments that are making use of fluid pages.  See the comments section for more details.

**<span style="text-decoration: underline;"><a href="https://www.peoplesoftmods.com/Development/PSM_2FA_POC.zip">CLICK HERE</a></span>** to download the project. Unzip the file and import the project into app designer.

Login to PeopleSoft and add the following section to the Signon PeopleCode.

<a href="/assets/images/2016/06/SignOnPeopleCodeFunctions.png" rel="attachment wp-att-148"><img class="alignnone size-full wp-image-148" src="/assets/images/2016/06/SignOnPeopleCodeFunctions.png" alt="Sign On PeopleCode" width="1152" height="648" srcset="/assets/images/2016/06/SignOnPeopleCodeFunctions.png 1152w, /assets/images/2016/06/SignOnPeopleCodeFunctions-300x169.png 300w, /assets/images/2016/06/SignOnPeopleCodeFunctions-768x432.png 768w, /assets/images/2016/06/SignOnPeopleCodeFunctions-1024x576.png 1024w, /assets/images/2016/06/SignOnPeopleCodeFunctions-676x380.png 676w" sizes="(max-width: 1152px) 100vw, 1152px" /></a>

Go to the web profile configuration and set the _Signon Result Doc Page_ field to _signonresultdocredirect.html_.

<a href="/assets/images/2016/06/WebProfileConfiguration.png" rel="attachment wp-att-149"><img class="alignnone size-full wp-image-149" src="/assets/images/2016/06/WebProfileConfiguration.png" alt="Web Profile Configuration" width="1152" height="885" srcset="/assets/images/2016/06/WebProfileConfiguration.png 1152w, /assets/images/2016/06/WebProfileConfiguration-300x230.png 300w, /assets/images/2016/06/WebProfileConfiguration-768x590.png 768w, /assets/images/2016/06/WebProfileConfiguration-1024x787.png 1024w, /assets/images/2016/06/WebProfileConfiguration-495x380.png 495w" sizes="(max-width: 1152px) 100vw, 1152px" /></a>

Bounce the web and app servers and log  back into PeopleSoft.  You should see the following page.

<a href="/assets/images/2016/06/2FAPage1.png" rel="attachment wp-att-150"><img class="alignnone size-full wp-image-150" src="/assets/images/2016/06/2FAPage1.png" alt="Two-Factor Authentication Page" width="368" height="249" srcset="/assets/images/2016/06/2FAPage1.png 368w, /assets/images/2016/06/2FAPage1-300x203.png 300w" sizes="(max-width: 368px) 100vw, 368px" /></a>

The email functionality will work if the SMTP settings are properly setup in your environment. The provided project does not include the necessary objects and code for the SMS functionality to work.  Nonetheless, you can select either delivery option. Click OK on the popup prompt.

<a href="/assets/images/2016/06/2FAPage2-1.png" rel="attachment wp-att-155"><img class="alignnone size-full wp-image-155" src="/assets/images/2016/06/2FAPage2-1.png" alt="Two-Factor Authentication Page" width="1259" height="387" srcset="/assets/images/2016/06/2FAPage2-1.png 1259w, /assets/images/2016/06/2FAPage2-1-300x92.png 300w, /assets/images/2016/06/2FAPage2-1-768x236.png 768w, /assets/images/2016/06/2FAPage2-1-1024x315.png 1024w, /assets/images/2016/06/2FAPage2-1-1180x363.png 1180w" sizes="(max-width: 1259px) 100vw, 1259px" /></a>

Since this is just a proof-of-concept, the application will automatically populate the code into the input box.  This is the same code that the user should receive to their email or cell phone. Once you are able to ensure that you can successfully send email and SMS messages, then the auto-populate functionality should be replaced with the user having to input the code themselves. You can check the _Remember This Device_ box if you do not want to have to do this process again on subsequent logins.

<a href="/assets/images/2016/06/2FAPage3-1.png" rel="attachment wp-att-156"><img class="alignnone size-full wp-image-156" src="/assets/images/2016/06/2FAPage3-1.png" alt="Two-Factor Authentication Page" width="374" height="328" srcset="/assets/images/2016/06/2FAPage3-1.png 374w, /assets/images/2016/06/2FAPage3-1-300x263.png 300w" sizes="(max-width: 374px) 100vw, 374px" /></a>

If you leave this box unchecked, then you will have to do this process again if you close the browser session and login again.  Clicking the _OK_ button should take you to the homepage and grant you full access into the system.

<span style="color: #ff0000;">Important Security Considerations:</span>

While this project may serve as a solid proof-of-concept of how you can implement 2FA in your PeopleSoft application, there are some things that I would like to point out.  The first thing is that using email is not a very secure avenue to deliver the time sensitive codes.  This is because, much like username and password, a user&#8217;s email account is merely something the user knows (email address and password).  It is much more secure to deliver the time sensitive codes to a physical device that the user has like their cell phone or a security token (not demonstrated). The second thing is that using the &#8220;Remember This Device&#8221; functionality makes this 2FA solution less secure.  This is because if a &#8220;remembered&#8221; device gets stolen, then it will potentially allow for an impostor to gain unauthorized access to another user&#8217;s account with just their username and password.  It is much more secure to delete the 2FA token at the end of every session.  To delete the 2FA token at the end of every session, just add the following cookie rule to the web profile:

<a href="/assets/images/2016/06/Delete_Token.png" rel="attachment wp-att-183"><img class="alignnone size-full wp-image-183" src="/assets/images/2016/06/Delete_Token.png" alt="Delete 2FA Token On Logout" width="946" height="500" srcset="/assets/images/2016/06/Delete_Token.png 946w, /assets/images/2016/06/Delete_Token-300x159.png 300w, /assets/images/2016/06/Delete_Token-768x406.png 768w, /assets/images/2016/06/Delete_Token-719x380.png 719w" sizes="(max-width: 946px) 100vw, 946px" /></a>

&nbsp;