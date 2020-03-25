---
id: 1352
title: Run Dynamic Code with Application Class Tester
date: 2020-03-22T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1352
permalink: /tips-and-tricks/run-dynamic-code-with-application-class-tester/
tags:
  - Security
  - Metadata API
categories:
  - Tips and Tricks
---

I previously did a post that documented a utility I created that allows you to 
[Compile and Run PeopleCode Online](https://www.peoplesoftmods.com/utilities/compile-and-run-peoplecode-online/).
This was a neat 
tool, but it required importing an App Designer project to be able to achieve the functionality of 
running dynamic PeopleCode in the PIA.  It would be much better to be able to run dynamic PeopleCode in the PIA using 
delivered tooling.  Well it turns out that you actually can do this by making use of the Application Class Tester 
and a delivered Application Class.  Combining these two PeopleSoft deliverables allows for creating and running custom 
Application Class PeopleCode all from the PIA.  

Check out the video demonstration on how to do this:

<iframe 
width="560" 
height="315" src="https://www.youtube.com/embed/5xOOtwjX58M" 
frameborder="0" 
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" 
allowfullscreen>
</iframe>  

* * *

Being able to run dynamic PeopleCode from the PIA is a bit of a double-edged sword.  I think it is quite 
obvious the security implications that could occur if this technique were abused by a malicious user. On the other hand, 
this technique could be used by a savvy developer to get out of a pinch.  With all things considered, I think it would be best to not 
allow any access to the Application Class Tester unless it is absolutely needed.