---
id: 185
title: Using the Event Mapping Framework to Enforce Two-Factor Authentication
date: 2016-06-25T20:00:57+00:00
guid: https://www.peoplesoftmods.com/?p=185
permalink: /2fa/using-the-event-mapping-framework-to-enforce-two-factor-authentication/
categories:
  - Event Mapping
  - Two-Factor Authentication
---
The Event Mapping Framework is a new functionality introduced in PeopleTools 8.55.  The framework provides a way to run custom code on delivered components without having to modify the delivered objects.  I am going to demonstrate how the Event Mapping Framework can be used to enforce two-factor authentication (2FA) by mapping application class PeopleCode to component events.  I have provided a proof-of-concept project that demonstrates this functionality.

<!--more-->

This project will add the ability to do a conditional redirect before a component is loaded. Before the pre-build event of the component is fired, the custom code will check if the user has done 2FA for the session. The code will redirect the user to the 2FA page if they have not done 2FA for the session, otherwise the component will load like normal.  The flow will go from this:

<a href="/assets/images/2016/06/Original_Logic-1.png" rel="attachment wp-att-205"><img class="alignnone size-full wp-image-205" src="/assets/images/2016/06/Original_Logic-1.png" alt="Original Logic" width="411" height="86" srcset="/assets/images/2016/06/Original_Logic-1.png 411w, /assets/images/2016/06/Original_Logic-1-300x63.png 300w" sizes="(max-width: 411px) 100vw, 411px" /></a>

To this:

<a href="/assets/images/2016/06/Redirect_Logic-1.png" rel="attachment wp-att-206"><img class="alignnone size-full wp-image-206" src="/assets/images/2016/06/Redirect_Logic-1.png" alt="Redirect Logic" width="440" height="535" srcset="/assets/images/2016/06/Redirect_Logic-1.png 440w, /assets/images/2016/06/Redirect_Logic-1-247x300.png 247w, /assets/images/2016/06/Redirect_Logic-1-313x380.png 313w" sizes="(max-width: 440px) 100vw, 440px" /></a>

**<span style="text-decoration: underline;"><a href="/assets/downloads/PSM_2FA_EVENT_MAPPING_POC.zip">CLICK HERE</a></span>** to download the project. Unzip the file and import the project into app designer.

Login to PeopleSoft and navigate to _Main Menu &#8211; > PeopleTools -> Portal -> Related Content Service -> Manage Related Content Service_.  Click on the _Event Mapping_ tab.

<a href="/assets/images/2016/06/1-Manage-Related-Content-Service.png" rel="attachment wp-att-186"><img class="alignnone size-full wp-image-186" src="/assets/images/2016/06/1-Manage-Related-Content-Service.png" alt="Manage Related Content Service" width="764" height="434" srcset="/assets/images/2016/06/1-Manage-Related-Content-Service.png 764w, /assets/images/2016/06/1-Manage-Related-Content-Service-300x170.png 300w, /assets/images/2016/06/1-Manage-Related-Content-Service-669x380.png 669w" sizes="(max-width: 764px) 100vw, 764px" /></a>

Click _Map the event of the Application pages_ link.

<a href="/assets/images/2016/06/2-Map-Event-of-App-Package.png" rel="attachment wp-att-187"><img class="alignnone size-full wp-image-187" src="/assets/images/2016/06/2-Map-Event-of-App-Package.png" alt="Map Event of App Package" width="776" height="259" srcset="/assets/images/2016/06/2-Map-Event-of-App-Package.png 776w, /assets/images/2016/06/2-Map-Event-of-App-Package-300x100.png 300w, /assets/images/2016/06/2-Map-Event-of-App-Package-768x256.png 768w" sizes="(max-width: 776px) 100vw, 776px" /></a>

For this demonstration, I would like to enforce 2FA on the _Change My Password_ page.  So select the _Change My Password_ content reference.

<a href="/assets/images/2016/06/3-Change-My-Password-CREF.png" rel="attachment wp-att-188"><img class="alignnone size-full wp-image-188" src="/assets/images/2016/06/3-Change-My-Password-CREF.png" alt="Change My Password CREF" width="772" height="828" srcset="/assets/images/2016/06/3-Change-My-Password-CREF.png 772w, /assets/images/2016/06/3-Change-My-Password-CREF-280x300.png 280w, /assets/images/2016/06/3-Change-My-Password-CREF-768x824.png 768w, /assets/images/2016/06/3-Change-My-Password-CREF-354x380.png 354w" sizes="(max-width: 772px) 100vw, 772px" /></a>

Next, you need to assign the related content definition (the definition was imported from the provided project) to the _Pre Build_ event on the component. Select _Pre Process_ for the processing sequence.  This means the custom application class will fire before any code that would be defined in the Pre Build event of the component.  Click save.

<a href="/assets/images/2016/06/4-Assign-Related-Content.png" rel="attachment wp-att-189"><img class="alignnone size-full wp-image-189" src="/assets/images/2016/06/4-Assign-Related-Content.png" alt="Assign Related Content" width="845" height="604" srcset="/assets/images/2016/06/4-Assign-Related-Content.png 845w, /assets/images/2016/06/4-Assign-Related-Content-300x214.png 300w, /assets/images/2016/06/4-Assign-Related-Content-768x549.png 768w, /assets/images/2016/06/4-Assign-Related-Content-532x380.png 532w" sizes="(max-width: 845px) 100vw, 845px" /></a>

Now navigate to the _Change My Password_ page.

<a href="/assets/images/2016/06/5-Change-My-Password-Link.png" rel="attachment wp-att-190"><img class="alignnone size-full wp-image-190" src="/assets/images/2016/06/5-Change-My-Password-Link.png" alt="Change My Password Link" width="457" height="749" srcset="/assets/images/2016/06/5-Change-My-Password-Link.png 457w, /assets/images/2016/06/5-Change-My-Password-Link-183x300.png 183w, /assets/images/2016/06/5-Change-My-Password-Link-232x380.png 232w" sizes="(max-width: 457px) 100vw, 457px" /></a>

And you should be prompted for 2FA at this point.

<a href="/assets/images/2016/06/6-2FA-Page.png" rel="attachment wp-att-191"><img class="alignnone size-full wp-image-191" src="/assets/images/2016/06/6-2FA-Page.png" alt="2FA Page" width="380" height="389" srcset="/assets/images/2016/06/6-2FA-Page.png 380w, /assets/images/2016/06/6-2FA-Page-293x300.png 293w, /assets/images/2016/06/6-2FA-Page-371x380.png 371w" sizes="(max-width: 380px) 100vw, 380px" /></a>

Note: The SMS option in the project is merely a placeholder.  I did a post on [how to send SMS text messages in PeopleSoft](https://www.peoplesoftmods.com/tips-and-tricks/sending-sms-text-messages-in-peoplesoft/) if you are interested in implementing this functionality.

After you perform 2FA and click OK, you should be redirected to the _Change My Password_ page.

<a href="/assets/images/2016/06/7-Change-My-Password.png" rel="attachment wp-att-192"><img class="alignnone size-full wp-image-192" src="/assets/images/2016/06/7-Change-My-Password.png" alt="Change My Password" width="554" height="361" srcset="/assets/images/2016/06/7-Change-My-Password.png 554w, /assets/images/2016/06/7-Change-My-Password-300x195.png 300w" sizes="(max-width: 554px) 100vw, 554px" /></a>

&nbsp;

With the Event Mapping Framework, the code to check if a user needs to perform 2FA can easily be applied to Pre Build events on components across the entire application.  The great thing is that the code is being fired in a custom event.  This means that we are able to achieve this customization of enforcing 2FA at the component level without actually having to &#8220;customize&#8221; the delivered components.

[I made a post here](https://www.peoplesoftmods.com/2fa/google-authenticator-in-peoplesoft/) that demonstrates how Google Authenticator can be implemented with the event mapping framework to enforce two-step verification at the component level in PeopleSoft.