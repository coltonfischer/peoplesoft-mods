---
id: 1353
title: PS-SwitchToken Bookmarklet Generator
date: 2020-03-25T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1353
permalink: /utilities/ps-switchtoken-generator/
tags:
  - SwitchUser
categories:
  - Utilities
---

PS-SwitchToken is a [JavaScript program](https://github.com/coltonfischer/ps-switchtoken) that allows 
you to seamlessly switch users during a PeopleSoft session.

Use the form below to generate a PS-SwitchToken bookmarklet for your PeopleSoft environment.  Input a title for 
the bookmarklet and the Local Node Password and then click Generate.

<script src="/assets/js/ps-switchtoken-generator.js"></script>
<form>
  <fieldset>
    Bookmark Title: <input  id="title" type="text" size="30" value="PS-SwitchToken"><br>
    Node Password: <input id="password" type="password" value="PS"><br>
  </fieldset>
  <a onclick="generateBookmarklet();" class="btn btn--info">Generate</a>
</form>
<b>Generated Bookmark: </b><a id="bookmarklet" href=""></a>

Drag the generated link into your browser bookmark bar and invoke it from a PeopleSoft page in your environment.  
You will get a prompt to input the User ID that you want to switch to:

[0]: /assets/images/2020/03/PS-SwitchToken.png
[![PS-SwitchToken][0]][0]

After inputting a valid User ID and clicking OK, the page will refresh and you will be logged in as the new user.

**Notice** This utility may not work for all PeopleSoft environments.  Check 
out the [GitHub Repository](https://github.com/coltonfischer/ps-switchtoken) for additional configuration 
details and troubleshooting tips.
{: .notice--warning}
