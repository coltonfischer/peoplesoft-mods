---
id: 1239
title: Response Manipulation with Portal Custodian
date: 2018-01-03T17:44:34+00:00
guid: http://www.peoplesoftmods.com/?p=1239
permalink: /psadmin/response-manipulation-with-portal-custodian/
tags:
  - Web Server
  - Servlet
categories:
  - PeopleSoft Administration
  - Tips and Tricks
---

What in the world is the Portal Custodian? I asked myself this very question when I came across a delivered file named _portalCustion.xml_ on the PeopleSoft web server.
The Portal Custodian is an undocumented functionality that allows for regular expression pattern matching replacements on the portal content served by the web server.
The portal content is the “wrapper” that the _psp_ servlet puts around the page content. This means that we have the ability to modify the contents within the portal header
and footer before the client receives the response from the web server. I discovered and tested this functionality in a PeopleSoft application running PeopleTools 8.56,
but it is quite likely that this feature exists in older Tools releases. In this post, I will walk through how we can use this interesting feature to manipulate response 
data.

The Portal Custodian functionality operates off of a configuration file on the web server named _portalCustodian.xml_. This file can be found in the following directory:

`%PIA_HOME%\webserv\domain\applications\peoplesoft\PORTAL.war\WEB-INF\psftdocs\ps`

The XML file contains a list of actions for the Portal Custodian. Each action represents a replacement that the Portal Custodian should perform. Each action contains 
four parameters:

  * **name** – A hardcoded value set to `ReplaceAll`

  * **contenturlpattern** – A regular expression pattern that, if found in the URL, will prompt the Portal Custodian to attempt string replacements in the response

  * **pattern** - A regular expression pattern that, if found in the response, will be replaced with the specified _replacewith_ string value

  * **replacewith** – A string value that will replace all of the matched _patterns_

The behavior of the Portal Custodian seems to indicate that the code makes use of Java’s _replaceAll_ method to perform the replacements on the responses. So this means 
that **all** occurrences of the _pattern_ will get replaced with the _replaceWith_ string.

Here is an example action that will perform a script injection on each response served by the _psp_ servlet. The script being injected will log a “Hello” message to the 
console on each page load.

```xml
<action>
  <name>ReplaceAll</name>
  <contenturlpattern>.*</contenturlpattern>
  <pattern>&lt;\/html&gt;</pattern>
  <replacewith>&lt;script&gt;console.log("Hello");&lt;/script&gt;$0</replacewith>
</action>
```

In the above example, the greater than and less than signs within the _pattern_ and _replacewith_ tags needed to be encoded for the action to work. After adding an action, 
a web server restart is required.