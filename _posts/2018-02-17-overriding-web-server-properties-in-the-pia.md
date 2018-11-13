---
id: 1303
title: Overriding Web Server Properties in the PIA
date: 2018-02-17T11:16:15+00:00
guid: http://www.peoplesoftmods.com/?p=1303
permalink: /psadmin/overriding-web-server-properties-in-the-pia/
tags:
  - Web Profile
  - Sign In
  - HTML
  - JavaScript
categories:
  - PeopleSoft Administration
  - Tips and Tricks
---
A while back I did a tutorial on how to define your own web profile custom properties. In [that post](/psadmin/defining-your-own-web-profile-custom-properties/) I demonstrated how custom meta-HTML variables can be defined on the web profile that can then be used in the server's static HTML files. This technique is good for providing an easy way to manage the web server properties in the PIA. Something that I did not mention in that post is that we can use the web profile custom properties page in the PIA to manage (and override) existing web server properties defined on the server. I have had good experiences using this technique to override delivered web server properties defined in the various .properties files on the web server. An example use case of this technique is to override properties in the text.properties file to achieve JavaScript injection on the PeopleSoft sign in page. In this post, I will demonstrate how we can use this technique so that we can have a custom sign in experience without having to customize delivered files.

The _text.properties_ file holds key-value pairs containing messages that get displayed on the various HTML files on the web server. The variable of interest for performing JavaScript injection on the sign in page is _#146_.

[1]: /assets/images/2018/02/TextProperties.png
[![Text Properties][1]][1]

This property is used for copyright text that gets displayed at the bottom of the _signin.html_ file. The reason I like to use this variable for JavaScript injection on the sign in page is because of the way that it is embedded in the _signin.html_ file.

[2]: /assets/images/2018/02/SignInHTML.png
[![Sign In HTML][2]][2]

Since the `<%=146%>` meta-tag is directly in between other HTML tags, I can override this variable with a custom script that will perform the customizations to the sign in page. This allows me to easily (ab)use property _#146_ as a built-in hook that will allow for a tailored sign in page that is customization-free.

Managing custom property _#146_ via the _text.properties_ file is undesirable as I want the ability to change the injected script without having to have access to the web server file system. Fortunately, we are able to use the Custom Properties tab on the Web Profile Configuration page in the PIA to override the value for property _#146_. We do not need to worry about the value defined in the _text.properties_ file as the value specified on the web profile will take precedence. You will simply need to set the Property Name to _146_, Validation Type to _String_ and Property Value to the custom script to modify the _signin.html_ page. As you can see from the picture below, you can append the script to the desired footer text so that you do not lose out on the message from getting displayed at the bottom of the sign in page.

[3]: /assets/images/2018/02/WebProfCustProp.png
[![Custom Property][3]][3]

After making this change and restarting the web server, you will see that your custom value in property _#146_ is injected in the footer of the sign in page. Here is some sample JavaScript that can be used to target and modify various elements on the _signin.html_ file (Note: Tested in 8.56.03).

<script src="https://gist.github.com/coltonfischer/ce68af143405208dab5e97c1f27e680d.js"></script>

Here is how the sign in page looks from storing the above fragment in the web profile custom property:

[4]: /assets/images/2018/02/CustomSignIn.png
[![Custom Sign In][4]][4]

The injected script performed various alterations to the sign in page. The script changed field labels, added an additional input field, and set the footer text with a dynamic year. With the ability to inject custom JavaScript into the sign in page, the possible sign in page transformations are practically endless.