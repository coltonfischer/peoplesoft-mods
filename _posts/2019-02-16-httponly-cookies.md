---
id: 1345
title: Controlling HttpOnly Cookies
date: 2019-02-16
guid: https://www.peoplesoftmods.com/?p=1345
permalink: /psadmin/controlling-httponly-cookies/
tags:
  - JavaScript
  - Web Profile
categories:
  - PeopleSoft Administration
---

Browser cookies served from PeopleSoft are marked as HttpOnly by default starting in PeopleTools 8.57.  HttpOnly cookies prevent client side scripts 
from accessing the cookie.  There are many scenarios where you may want to access a PeopleSoft cookie via JavaScript in the browser.  You can 
disable the HttpOnly flag for a PeopleSoft cookie by specifying the cookie name on the `Browser Cookie Rules` grid on the `Cookie Rules` tab of 
the Web Profile. 

[1]: /assets/images/2019/02/Web_Profile.png
[![Web Profile Cookie Rules][1]][1]

Using the Web Profile configuration for disabling HttpOnly flags for cookies works well, but there are scenarios where the developer may want to 
control whether the HttpOnly flag is set for a cookie.  Unfortunately, the delivered `Cookie` class used for creating cookies in PeopleCode does 
not provide a way to set the HttpOnly flag.  If a developer wants to disable the HttpOnly flag for a cookie via code, then they can use the 
`SetHeader` method of the `%Response` class to create the cookie manually.

```java
   %Response.SetHeader("Set-Cookie", "MyCookie=123; path=/");
```

This results in the cookie being marked as non-HttpOnly in the browser.

[2]: /assets/images/2019/02/Cookie.png
[![Non-HttpOnly Cookie][2]][2]

* * *

PeopleSoft cookies being served as HttpOnly by default is a great security enhancement.  However, this seemingly small enhancement can cause some severe 
issues for customers upgrading to 8.57 that require cookies to be JavaScript accessible.  Understanding the demonstrated techniques to override this delivered 
behavior will surely help resolve these potential post-upgrade issues.