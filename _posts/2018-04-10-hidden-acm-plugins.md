---
id: 1341
title: Hidden ACM Plugins
date: 2018-04-10T06:36:32+00:00
guid: http://www.peoplesoftmods.com/?p=1341
permalink: /psadmin/hidden-acm-plugins/
categories:
  - PeopleSoft Administration
  - Tips and Tricks
---
In <a href="http://psadmin.io/2018/02/16/120-let-me-introduce-you-to-effdt/" target="_blank">episode #120</a> of the PeopleSoft Administrator Podcast, there is a discussion on the deltas in the PTEM_CONFIG App Package from 8.55 to 8.56.  In 8.56 there appears to be a ton of delivered ACM Plugin App Classes, but as Dan explains, these App Classes are completely empty.  This is a major tease because there are some really interesting ACM Plugin App Class names such as PTScriptExecutor and PTWebServerConfigUpdate that have no implementation code in 8.56.

<!--more-->

I made an interesting discovery when I did a _Find In_ operation in App Designer for these ACM Plugin App Classes that appeared to have no implementation.  What I found is that these classes actually do have an implementation, but we cannot see them when we open the App Class definition.  This may be a new PeopleTools “feature” that allows the internal PeopleTools team to have ACM Plugins that they do not want to support customer usage of.

To get the implementation code of a “hidden” ACM Plugin, you will need to open App Designer and go to Edit > Find In&#8230;.  If you search for the ACM Plugin App Class name (Ex: PTScriptExecutor), you will get a result that points to the PTEM_CONFIG:PTScriptExecutor App Class.  However, when you click the reference to open the App Class PeopleCode, you are greeted with a blank class.  The trick is to check the _Save PeopleCode to file_ checkbox when performing the _Find In_ search.

[<img src="/assets/images/2018/03/Find_In.png" alt="Find In..." />](/assets/images/2018/03/Find_In.png)

This will save the entire ACM Plugin App Class PeopleCode program to the file that you specify.  From here you can copy the code into an App Class within your own PTEM_CONFIG* App Package to make use of the plugin.

Alternatively, you can use the following query to expose all of the &#8220;hidden&#8221; PeopleCode programs:

<pre>SELECT A.*
FROM PSPCMTXT A
WHERE NOT EXISTS (SELECT 1
 FROM PSPCMPROG B
 WHERE B.OBJECTVALUE1 = A.OBJECTVALUE1
 AND B.OBJECTVALUE2 = A.OBJECTVALUE2
 AND B.OBJECTVALUE3 = A.OBJECTVALUE3
 AND B.OBJECTVALUE4 = A.OBJECTVALUE4
 AND B.OBJECTVALUE5 = A.OBJECTVALUE5
 AND B.OBJECTVALUE6 = A.OBJECTVALUE6
 AND B.OBJECTVALUE7 = A.OBJECTVALUE7);</pre>

This query returns 56 rows in my 8.56.03 environment.  All of the rows are PeopleCode programs withing the PTEM_CONFIG Application Package.  I am not sure if these 56 PeopleCode programs were intentionally delivered in this manner, but this trick to expose the hidden programs may be something to keep in mind when you come across empty PeopleCode Programs in App Designer in the future.

<span style="color: #ff0000;"><strong>UPDATE (5/2/18)</strong></span>: The technique demonstrated in this post may not work if you have previously ran the _UPGPTHASH_ Application Engine program in your PeopleSoft environment.  It appears that this program is used to remove orphaned rows from the _PSPCMTXT_ table.  Check out <a href="https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=267952209929499&id=2361170.1" target="_blank">Doc ID 2361170.1</a> for more information.
