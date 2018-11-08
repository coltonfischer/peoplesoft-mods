---
id: 704
title: Sending SMS Text Messages in PeopleSoft
date: 2016-10-05T16:08:27+00:00
guid: http://www.peoplesoftmods.com/?p=704
permalink: /tips-and-tricks/sending-sms-text-messages-in-peoplesoft/
categories:
  - Tips and Tricks
---
In this post I will provide a step-by-step tutorial on how to send SMS text messages in PeopleSoft.  I will be consuming Nexmo’s SMS API to send SMS messages.  There are many SMS API services similar to Nexmo and there is no particular reason that I chose Nexmo over the other providers for this tutorial.  I have some experience with using other providers and the quality of service is comparable across the board. There will be four main steps in this tutorial: creating a Nexmo account, importing Nexmo SSL certificates, importing custom objects, and testing the service.

<!--more-->

**Creating a Nexmo Account**

The first thing that you will need to do is to create a free account with Nexmo.  <a href="https://dashboard.nexmo.com/sign-up" target="_blank">Navigate to their site</a> and create a new account.

[<img class="alignnone size-full wp-image-705" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1.png" alt="Picture1" width="1210" height="734" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1.png 1210w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1-300x182.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1-768x466.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1-1024x621.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1-626x380.png 626w" sizes="(max-width: 1210px) 100vw, 1210px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture1.png)

Be sure to enter a valid phone number for the setup.  An SMS with a code will be sent to the mobile phone number that you enter.  Input the code that you received on the next screen to get access to your account.

Once you are logged in, navigate to the “Getting Started” tab and make note of your API key, API secret, and test number. Then Click on the “your numbers” link to obtain your sender ID.

[<img class="alignnone size-full wp-image-707" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3.png" alt="Picture3" width="1183" height="976" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3.png 1183w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3-300x248.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3-768x634.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3-1024x845.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3-461x380.png 461w" sizes="(max-width: 1183px) 100vw, 1183px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture3.png)

Make note of the sender ID that is assigned to your account. This will be the number that the SMS messages will come from.

[<img class="alignnone size-full wp-image-708" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture4.png" alt="Picture4" width="974" height="398" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture4.png 974w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture4-300x123.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture4-768x314.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture4-930x380.png 930w" sizes="(max-width: 974px) 100vw, 974px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture4.png)

**Importing Nexmo SSL Certificates**

In this step you will need to download Nexmo’s SSL certificates and import them into the PeopleSoft PSKeyManager.  The primary URL that you will need to obtain the SSL certificates from is https://nexmo.com.  You will need to download the root and intermediate certificates from this URL.

[<img class="alignnone size-full wp-image-709" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture5.png" alt="Picture5" width="776" height="616" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture5.png 776w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture5-300x238.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture5-768x610.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture5-479x380.png 479w" sizes="(max-width: 776px) 100vw, 776px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture5.png)

Then you will need to import the root and intermediate certificates into PSKeyManager.  The whole process of importing third-party SSL certificates into PeopleSoft is already nicely <a href="http://nirajpatil-psadmin.blogspot.com/2012/09/psoft-third-party-integration-using.html" target="_blank">documented here by Niraj Patil</a>, so I suggest following the steps in that post to import the certificates. Alternatively, you can [click here to download a Word doc version with pictures](http://www.peoplesoftmods.com/Development/Integration_with_a_3rd_Party_utilizing_Digital_Certificates_v4.doc).

**Importing Custom Objects**

I have created an app designer project that contains all of the necessary objects to invoke the SMS API web service via Integration Broker in PeopleSoft. [Click here to download the app designer project](http://www.peoplesoftmods.com/Development/PSM_NEXMO.zip).  After importing the project into app designer, you can login to the PIA to view the newly imported service operation that will be used for sending SMS messages.  The service operation is located at PeopleTools->Integration Broker->Integration Setup->Service Operations. The service operation is named “SENDSMS_GET”. You can click on the “View Message” link to see the document template that will get populated when invoking the service.

[<img class="alignnone size-full wp-image-710" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture6.png" alt="Picture6" width="849" height="482" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture6.png 849w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture6-300x170.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture6-768x436.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture6-669x380.png 669w" sizes="(max-width: 849px) 100vw, 849px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture6.png)

The document consists of the elements that correlate to the information that you obtained in the first step of this tutorial.

[<img class="alignnone size-full wp-image-711" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture7.png" alt="Picture7" width="434" height="447" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture7.png 434w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture7-291x300.png 291w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture7-369x380.png 369w" sizes="(max-width: 434px) 100vw, 434px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture7.png)

**Testing the Service**

Before attempting to execute the web service via PeopleCode, it would be a good idea to test the service using the built in Service Opertaion Tester utility in the PIA first.  To do this, you will need to navigate to PeopleTools->Integration Broker->Service Utilities->Service Operation Tester.  Search for the “SENDSMS_GET” service operation. After you select the service operation, you will need to click the “Populate Document Template” link to input your Nexmo account details.

[<img class="alignnone size-full wp-image-713" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture9.png" alt="Picture9" width="599" height="383" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture9.png 599w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture9-300x192.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture9-594x380.png 594w" sizes="(max-width: 599px) 100vw, 599px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture9.png)

Input  your Nexmo account information by clicking on the corresponding element name for each piece of data and typing in the value.  The “text” field is the body of the SMS message, so you can put anything for this value.

[<img class="alignnone size-full wp-image-714" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture10.png" alt="Picture10" width="513" height="428" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture10.png 513w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture10-300x250.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture10-455x380.png 455w" sizes="(max-width: 513px) 100vw, 513px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture10.png)

After clicking the return button, you will be taken back the previous screen.  Now you will need to click the “Invoke Operation” button to fire the Integration Broker message to send an SMS.

[<img class="alignnone size-full wp-image-715" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture11.png" alt="Picture11" width="673" height="465" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture11.png 673w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture11-300x207.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture11-550x380.png 550w" sizes="(max-width: 673px) 100vw, 673px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture11.png)

If the service operation is not working, then you will receive a popup error at this point.  If the service operation is working, then you should receive an SMS message on the number that you inputted in the “to” field.  The JSON response from Nexmo should also appear on the screen at this point.

If you are able to successfully invoke the service operation using the testing utility, then you can now confidently invoke the service operation using PeopleCode.  Here is some code that you can use to send SMS messages from PeopleCode.  I have also attached a copy of this code as a text file in the app designer project linked above.

[<img class="alignnone size-full wp-image-716" src="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture12.png" alt="Picture12" width="736" height="412" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture12.png 736w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture12-300x168.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture12-679x380.png 679w" sizes="(max-width: 736px) 100vw, 736px" />](http://www.peoplesoftmods.com/wp-content/uploads/2016/10/Picture12.png)

I would recommend extending this sample code to be more robust by reading in the Nexmo account details from a setup table in the database.  It would also be a good idea to parse the response message and implement some sort of error handling.  Putting this code in a function or app class method that accepts phone number and message body parameters will make it easy to integrate anywhere in your application.

* * *

Sending SMS text messages from PeopleSoft using Nexmo&#8217;s SMS API is really simple after you perform this initial setup.  My main use case for sending SMS messages from PeopleSoft is to send TOTPs to user&#8217;s cell phones for two-factor authentication. However, I believe there are many other ways that this functionality can be leveraged in PeopleSoft to enhance the application.