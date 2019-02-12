---
id: 829
title: POC Google Authenticator Project
date: 2017-02-02T07:41:57+00:00
guid: https://www.peoplesoftmods.com/?p=829
permalink: /2fa/poc-google-authenticator-project/
categories:
  - Event Mapping
  - Two-Factor Authentication
---
A recent comment I received made me go back and revisit my “[Implementing Google Authenticator in PeopleSoft](https://www.peoplesoftmods.com/2fa/implementing-google-authenticator-in-peoplesoft/)” post where I discuss the code involved to get Google Authenticator working in PeopleSoft.  Revisiting this post made me realize that I never actually shared the source App Designer project with the community.  I would like to use this post to share the plug and play POC project that I use to enforce component-level 2FA with Google Authenticator in PeopleSoft applications.

<!--more-->

Disclaimer: I will admit that the design and logic of parts of this project are a little sketchy.  I created this POC project on the fly a while back with the intent of revisiting it to clean it up.  I began to shift my research efforts from component-level 2FA schemes to field-level 2FA schemes and I never took the time to revisit this project.  I can certainly create a better version of this project if there is interest.

With that disclaimer out of the way, this post will contain the steps to install the project and get it up and running.  If you would like more technical details of how the project works, then [check this post out](https://www.peoplesoftmods.com/2fa/implementing-google-authenticator-in-peoplesoft/).

Add [psGAuth.jar](https://www.peoplesoftmods.com/Development/psGAuth.jar) to class directory in your PS_HOME on the app server (_%PS_HOME%\class\psGAuth.jar_) and bounce the app server with a cache clear.

**<span style="color: #ff0000;">UPDATE</span>** 2/22/18: The provided JAR file does not function properly in PeopleTools 8.56 as it it dependent on an older version of  Apache Commons Codec.  See comments for more details.

**<span style="color: #ff0000;">UPDATE</span> **3/30/18:  Check out the post [Validating Time-Based One-Time Passwords](https://www.peoplesoftmods.com/2fa/validating-time-based-one-time-passwords/) if you do not want to have the custom Java class (psGAuth.jar) app server dependency.  That post demonstrates an alternative method to validating TOTPs that does not require an app server dependency.

Import the [App Designer project](https://www.peoplesoftmods.com/Development/PSM_GAUTH_POC.zip) into App Designer and build the project.

[<img class="alignnone size-full wp-image-830" src="/assets/images/2017/02/1_Import_Build_Project.png" alt="Import and Build Project" width="1039" height="779" srcset="/assets/images/2017/02/1_Import_Build_Project.png 1039w, /assets/images/2017/02/1_Import_Build_Project-300x225.png 300w, /assets/images/2017/02/1_Import_Build_Project-768x576.png 768w, /assets/images/2017/02/1_Import_Build_Project-1024x768.png 1024w, /assets/images/2017/02/1_Import_Build_Project-507x380.png 507w" sizes="(max-width: 1039px) 100vw, 1039px" />](/assets/images/2017/02/1_Import_Build_Project.png)

Login to the PIA and assign a user the provided PSM_GAUTH role.

[<img class="alignnone size-full wp-image-833" src="/assets/images/2017/02/2_Assign_Role.png" alt="Assign Role" width="1039" height="367" srcset="/assets/images/2017/02/2_Assign_Role.png 1039w, /assets/images/2017/02/2_Assign_Role-300x106.png 300w, /assets/images/2017/02/2_Assign_Role-768x271.png 768w, /assets/images/2017/02/2_Assign_Role-1024x362.png 1024w" sizes="(max-width: 1039px) 100vw, 1039px" />](/assets/images/2017/02/2_Assign_Role.png)

If you are on 8.55+ Tools, then you can make use of event mapping to achieve the 2FA functionality.  To do this, map the provided related content service named PSM\_ENFORCE\_GAUTH to the Pre Build event of a component that you want enable 2FA on.

[<img class="alignnone size-full wp-image-834" src="/assets/images/2017/02/3_Map_Event.png" alt="Map Event" width="1039" height="578" srcset="/assets/images/2017/02/3_Map_Event.png 1039w, /assets/images/2017/02/3_Map_Event-300x167.png 300w, /assets/images/2017/02/3_Map_Event-768x427.png 768w, /assets/images/2017/02/3_Map_Event-1024x570.png 1024w, /assets/images/2017/02/3_Map_Event-683x380.png 683w" sizes="(max-width: 1039px) 100vw, 1039px" />](/assets/images/2017/02/3_Map_Event.png)

If you are not yet on the 8.55 Tools, then you will have to customize the Pre Build event of the component that you want to enable 2FA on.  I highly recommend getting on 8.55 to avoid this type of customization, but you can paste the following code into the beginning of the component&#8217;s Pre Build event to enforce 2FA:

<pre>/* import GAuth */

import PSM_GAUTH_APP_PKG:EnforceGAuth;

/* create GAuth instance */

Local PSM_GAUTH_APP_PKG:EnforceGAuth &GAuth = create PSM_GAUTH_APP_PKG:EnforceGAuth();

/* run execute to check if 2FA is needed */

&GAuth.Execute();</pre>

For example:

[<img class="alignnone size-full wp-image-835" src="/assets/images/2017/02/4_Customize_Component.png" alt="Customize Component" width="830" height="221" srcset="/assets/images/2017/02/4_Customize_Component.png 830w, /assets/images/2017/02/4_Customize_Component-300x80.png 300w, /assets/images/2017/02/4_Customize_Component-768x204.png 768w" sizes="(max-width: 830px) 100vw, 830px" />](/assets/images/2017/02/4_Customize_Component.png)

Now when you navigate to the 2FA-enabled component as the user with the PSM_GAUTH role, then you should be prompted to do a one-time setup for Google Authenticator.  The next time you access a 2FA-enabled component, you will be prompted to input a Google Authenticator TOTP unless you possess a valid 2FA cookie (more on this below).

<span style="color: #ff0000;">Important Consideration:</span>

The code is written to issue a 2FA cookie after a successful Google Authenticator challenge. On subsequent requests to 2FA-enabled components, the code will not challenge a user for 2FA if they possess a valid 2FA cookie in their browser.  I applied this logic to the project to demonstrate a general idea of how you can go about satisfying potential usability requirements.  I strongly recommend that you evaluate this practice to determine if you want or need your system to allow for this sort of 2FA bypass mechanism.  If you decide to allow for the possession of a cookie to bypass 2FA, then please DO NOT rely on the way that it is coded in the project to achieve this functionality.

Included in this project are some objects to make the 2FA process more configurable.  This includes things like setup tables to house which PeopleSoft roles to enforce 2FA for and which IP address ranges should be trusted to bypass 2FA.  Similar to the 2FA cookie bypass mechanism I described above, I included these mechanisms to expose the idea of how make the 2FA process more accommodating to usability requirements.

If you are interesting in implementing this project feel free to download and give it a try, but please keep in mind that it is merely a proof of concept of how you can go about enforcing component-level 2FA.  Please feel free to ask any questions that you may have about this project.