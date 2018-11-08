---
id: 1076
title: Conditional Data Masking with Event Mapping
date: 2017-07-28T06:56:20+00:00
guid: http://www.peoplesoftmods.com/?p=1076
permalink: /emf/conditional-data-masking-with-event-mapping/
categories:
  - Data Masking
  - Event Mapping
---
Using Event Mapping to perform field level data masking is an idea that I have toyed with since the release of Event Mapping in PeopleTools 8.55.  Event Mapping is a desirable tool for field level data masking in PeopleSoft because it can allow for bolt on runtime application logic to determine if a user should have the ability to view a particular piece of data.  I am unfortunately not here to say that Event Mapping can deliver us a (much needed) data masking framework for PeopleSoft applications.  However, I have recently found that Event Mapping is capable of satisfying one particular field level data masking requirement that I threw at it.  In this post, I will be sharing a proof of concept project that I’ve used to perform conditional data masking on a read only data field within a delivered Fluid page.

<!--more-->

**<u>Project Overview and Requirements</u>**

My requirement is to mask the Social Security number that shows up on the Fluid Personal Details within Campus Solutions.

[<img class="alignnone size-full wp-image-1077" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before.png" alt="Personal Details Before" width="1476" height="586" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before.png 1476w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before-300x119.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before-768x305.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before-1024x407.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before-957x380.png 957w" sizes="(max-width: 1476px) 100vw, 1476px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-Before.png)

The field needs to get masked at component load time and it shall have the ability to be conditionally unmasked based on custom logic that evaluates the user’s session attributes.  The SSN field will get replaced with an image that provides an on click event.

[<img class="alignnone size-full wp-image-1078" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After.png" alt="Personal Details After" width="1399" height="606" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After.png 1399w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After-300x130.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After-768x333.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After-1024x444.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After-877x380.png 877w" sizes="(max-width: 1399px) 100vw, 1399px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-After.png)

The outcome of the click event is determined at component load time and it provides for three possible outcomes: Click-To-Deny, Click-To-View, or Click-To-Challenge.  Below I will explain these three scenarios and how they are achieved.

**Click-To-Deny:**

This scenario is for when you do not want the user to have the ability to see the unmasked SSN.  When the user clicks the lock icon, a modal pops up that tells the user they are unauthorized to view the data.

[<img class="alignnone size-full wp-image-1079" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny.png" alt="Click To Deny" width="1372" height="613" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny.png 1372w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny-300x134.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny-768x343.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny-1024x458.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny-851x380.png 851w" sizes="(max-width: 1372px) 100vw, 1372px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Deny.png)

This scenario is achieved by discarding the SSN value in the component buffer at component load time and injecting an onload script that contains the logic to display the popup modal on the onclick event.

**Click-To-View:**

This scenario is used for when you want to allow for the user to view the data, but you still want the data to be masked at runtime.  This is desirable in circumstances where you want to prevent unnecessary sensitive data leakage.  If the user is performing a transaction on the page that requires the knowledge of the SSN, then the user can simply click to view the SSN.  Otherwise, the SSN will never be exposed to the client.

For the Click-To-View functionality to work, the original SSN value is encrypted at component load time and is communicated to the client’s browser via an injected onload script.  If the user needs to view the masked data, the encrypted value is sent to an IScript in the onclick event that will validate the request, log the transaction details, and respond back with the decrypted value.

**Click-To-Challenge:**

This is by far the most interesting scenario as it exercises a field level multifactor authentication solution for PeopleSoft.  In this scenario, if a user wants to view the masked data, then they will be prompted for a secondary form of authentication on the click event.

[<img class="alignnone size-full wp-image-1080" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge.png" alt="Click To Challenge" width="1381" height="623" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge.png 1381w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge-300x135.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge-768x346.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge-1024x462.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge-842x380.png 842w" sizes="(max-width: 1381px) 100vw, 1381px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Click-To-Challenge.png)

The Click-To-Challenge functionality is achieved via a combination of the methodologies used in the Click-To-Deny and Click-To-View functionalities.   One difference is that the popup modal contains an input form to allow receipt of the user’s secondary form of authentication.  The other difference is in the call to the IScript on the form submit.  The IScript behaves identical to the Click-To-View IScript except that it also validates the user’s secondary form of authentication before responding back with the decrypted value.

**<u>Project Implementation</u>**

If you are interested in implementing this proof of concept solution for the Campus Community Fluid Personal Details page in your Campus Solutions application, then follow the steps below.

[**CLICK HERE**](http://www.peoplesoftmods.com/Development/PSM_DATA_MASK_EMF_POC.zip) to download App Designer Project and import it into app designer. Map the included Related Content Service using Event Mapping to the Fluid Personal Details CREF.

[<img class="alignnone size-full wp-image-1081" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-CREF.png" alt="Personal Details CREF" width="840" height="908" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-CREF.png 840w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-CREF-278x300.png 278w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-CREF-768x830.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-CREF-352x380.png 352w" sizes="(max-width: 840px) 100vw, 840px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Personal-Details-CREF.png)

Map the PSM\_MASK\_PROFILE_SSN service to the Post Build/Post Process event

[<img class="alignnone size-full wp-image-1082" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Map-Event.png" alt="Map Event" width="906" height="547" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Map-Event.png 906w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Map-Event-300x181.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Map-Event-768x464.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Map-Event-629x380.png 629w" sizes="(max-width: 906px) 100vw, 906px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Map-Event.png)

Last, set the permission for the Click-To-View IScript and the Click-To-Challenge IScript.

[<img class="alignnone size-full wp-image-1083" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Set-Permissions.png" alt="Set Permissions" width="969" height="461" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Set-Permissions.png 969w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Set-Permissions-300x143.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Set-Permissions-768x365.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Set-Permissions-799x380.png 799w" sizes="(max-width: 969px) 100vw, 969px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/07/Set-Permissions.png)

Note: You will need to implement a multi-factor authentication solution and extend the Click-To-Challenge IScript for it to serve its intended purpose.  I have documented how I’ve [implemented Google Authenticator in PeopleSoft](http://www.peoplesoftmods.com/2fa/implementing-google-authenticator-in-peoplesoft/) if you are interested in this form of secondary authentication.

It should be clear that this type of data masking solution will not work on every field in the application.  There are a couple of prerequisites that must be met in order for a solution like this to be considered.  This solution requires that the field to be masked must be read only and reside on a Fluid page.   Even then however, there still may be some nuances that will prevent this solution from working properly.  This demonstration was done on a Campus Solutions 9.2 application running PeopleTools 8.56.