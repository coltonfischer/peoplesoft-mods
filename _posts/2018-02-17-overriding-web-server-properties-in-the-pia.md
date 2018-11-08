---
id: 1303
title: Overriding Web Server Properties in the PIA
date: 2018-02-17T11:16:15+00:00
guid: http://www.peoplesoftmods.com/?p=1303
permalink: /psadmin/overriding-web-server-properties-in-the-pia/
categories:
  - PeopleSoft Administration
  - Tips and Tricks
---
A while back I did a tutorial on how to define your own web profile custom properties.  In [that post](http://www.peoplesoftmods.com/psadmin/defining-your-own-web-profile-custom-properties/) I demonstrated how custom meta-HTML variables can be defined on the web profile that can then be used in the server&#8217;s static HTML files.  This technique is good for providing an easy way to manage the web server properties in the PIA.  Something that I did not mention in that post is that we can use the web profile custom properties page in the PIA to manage (and override) existing web server properties defined on the server.  I have had good experiences using this technique to override delivered web server properties defined in the various .properties files on the web server.  An example use case of this technique is to override properties in the text.properties file to achieve JavaScript injection on the PeopleSoft sign in page.  In this post, I will demonstrate how we can use this technique so that we can have a custom sign in experience without having to customize delivered files.

<!--more-->

The text.properties file holds key-value pairs containing messages that get displayed on the various HTML files on the web server.  The variable of interest for performing JavaScript injection on the sign in page is #146.

[<img class="alignnone size-full wp-image-1304" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/TextProperties.png" alt="TextProperties" width="912" height="697" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/TextProperties.png 912w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/TextProperties-300x229.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/TextProperties-768x587.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/TextProperties-497x380.png 497w" sizes="(max-width: 912px) 100vw, 912px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/02/TextProperties.png)

This property is used for copyright text that gets displayed at the bottom of the signin.html file.  The reason I like to use this variable for JavaScript injection on the sign in page is because of the way that it is embedded in the signin.html file.

[<img class="alignnone size-full wp-image-1305" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/SignInHTML.png" alt="SignInHTML" width="1009" height="599" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/SignInHTML.png 1009w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/SignInHTML-300x178.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/SignInHTML-768x456.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/SignInHTML-640x380.png 640w" sizes="(max-width: 1009px) 100vw, 1009px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/02/SignInHTML.png)

Since the <%=146%> meta-tag is directly in between other HTML tags, I can override this variable with a custom script that will perform the customizations to the sign in page.  This allows me to easily (ab)use property #146 as a built-in hook that will allow for a tailored sign in page that is customization-free.

Managing custom property #146 via the text.properties file is undesirable as I want the ability to change the injected script without having to have access to the web server file system.  Fortunately, we are able to use the Custom Properties tab on the Web Profile Configuration page in the PIA to override the value for property #146.  We do not need to worry about the value defined in the text.properties file as the value specified on the web profile will take precedence.  You will simply need to set the Property Name to 146, Validation Type to String and Property Value to the custom script to modify the signin.html page.  As you can see from the picture below, you can append the script to the desired footer text so that you do not lose out on the message from getting displayed at the bottom of the sign in page.

[<img class="alignnone size-full wp-image-1306" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp.png" alt="WebProfCustProp" width="1187" height="976" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp.png 1187w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp-300x247.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp-768x631.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp-1024x842.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp-462x380.png 462w" sizes="(max-width: 1187px) 100vw, 1187px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/02/WebProfCustProp.png)

After making this change and restarting the web server, you will see that your custom value in property #146 is injected in the footer of the sign in page.  Here is some sample JavaScript that can be used to target and modify various elements on the signin.html file (Note: Tested in 8.56.03).



Here is how the sign in page looks from storing the above fragment in the web profile custom property:

[<img class="alignnone size-full wp-image-1308" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn.png" alt="CustomSignIn" width="1152" height="820" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn.png 1152w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn-300x214.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn-768x547.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn-1024x729.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn-534x380.png 534w" sizes="(max-width: 1152px) 100vw, 1152px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/02/CustomSignIn.png)

The injected script performed various alterations to the sign in page.  The script changed field labels, added an additional input field, and set the footer text with a dynamic year.  With the ability to inject custom JavaScript into the sign in page, the possible sign in page transformations are practically endless.