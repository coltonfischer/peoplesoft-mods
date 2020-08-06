---
id: 1358
title: Display iScripts in Fluid Nav Collections
date: 2020-08-05T00:56:55+00:00
guid: http://www.peoplesoftmods.com/?p=1358
permalink: /ux/display-iscripts-in-fluid-nav-collections/
tags:
  - iScripts
  - Navigation
  - JavaScript
categories:
  - User Experience
---

There is a pesky bug that has been a thorn in my side for quite some time.  The issue is the inability to use iScript based Content References in Fluid Navigation Collections.  Viewing an iScript CREF in a Fluid Nav Collection results in the left hand side navigation locking up.  This is a [documented bug](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2306226.1) that does not have a solution listed.

## Cause

I found the cause of this issue to be the absence of the `bLoadCompleted` JavaScript variable in the iScript HTML response content.  This variable appears in whats seems to be all of the HTML response content of Component based applications.  Unlike Components, iScript response content is entirely controlled by the developer.   This is great for the case of developing unaltered response content, but it is problematic in this scenario where the system navigation expects particular content to be present in the response.

## Solution

Following the supplied example in the bug documentation, consider the following iScript response:
```java
Function IScript_Test()  
   %Response.WriteLine("<p>Hello world</p>");  
End-Function;
```
If you were to create a CREF to represent this iScript function and add it to a Navigation Collection it would show up in the left hand navigation pane just fine when rendered in a Fluid Navigation Collection style Activity Guide.  However, once you select to view the iScript content in the Fluid Navigation Collection the navigation will lock up.

You can alter the response of the iScript as follows to prevent the navigation from locking up:
```java
Function IScript_Test()  
   %Response.WriteLine("<script>var bLoadCompleted = true;</script>");
   %Response.WriteLine("<p>Hello world</p>");  
End-Function;
```
## Conclusion

As I mentioned above, iScripts are great if your requirement is to have complete control over response content for the applications you build.  You can create applications with amazing user interfaces and performance with iScripts, but this comes at the cost of having to understand and ensure that the response content conforms to the requirements that the higher level system components expect.