---
id: 1335
title: Managing Large HTML Objects
date: 2018-03-10T10:44:26+00:00
guid: http://www.peoplesoftmods.com/?p=1335
permalink: /tips-and-tricks/managing-large-html-objects/
tags:
  - JavaScript
  - App Designer
  - Branding Framework
  - HTML
categories:
  - Tips and Tricks
---

I wanted to share a quick tip on managing third-party JavaScript libraries in HTML objects in PeopleTools. The tip is to manage these HTML objects in the Branding Framework in the PIA rather than using App Designer. I have found that App Designer does not play well with handling large JavaScript libraries as it tends to add newlines in the code after line lengths reach a certain point. Minified JavaScript libraries are notorious for having extremely long line lengths and this is an issue for App Designer.

For example, let’s say you wanted to store and serve the [minified jQuery v3.3.1](https://code.jquery.com/jquery-3.3.1.min.js) JavaScript library from a custom HTML object. You would copy and paste the JavaScript code into a new HTML object definition in App Designer and save the new definition.

[1]: /assets/images/2018/03/Save_App_Designer.png
[![Save App Designer][1]][1]

At this point you could locally serve the jQuery library by using `%Response.GetJavaScriptURL()`. This is when you will be greeted with an error similar to the following:

[2]: /assets/images/2018/03/Unexpected_Token.png
[![Unexpected Token][2]][2]

The error _Uncaught SyntaxError: Invalid or unexpected token_ is a result from a line of code in the JavaScript unexpectedly ending. This is due to the way that App Designer saved the HTML object.

If you want to successfully serve large JavaScript libraries from PeopleTools HTML objects, then you will need to use the Branding Framework in the PIA (`Main Menu > PeopleTools > Portal > Branding > Branding Objects`) to create the new HTML object to house the JavaScript code.

[3]: /assets/images/2018/03/Save_Branding_Framework.png
[![Save Branding Framework][3]][3]

I have found that the Branding Framework does not add newlines for long line lengths like App Designer does.

It is also very important to note that you should use the Branding Framework to **edit** these large HTML objects and not just for creating them. If you edit a large HTML object (created with the Branding Framework) in App Designer, the object will get corrupted when you save it. This will result in the _Unexpected Token_ error and you will have to recreate the HTML object in the Branding Framework.

This does not happen with every large JavaScript library, but rather only the ones that have long line lengths. However, I have been bit by this App Designer shortcoming so many times that I have gotten into the habit of always using the Branding Framework to manage large HTML objects to avoid potential headaches with using App Designer.