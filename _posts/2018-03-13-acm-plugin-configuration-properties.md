---
id: 1348
title: ACM Plugin Configuration Properties
date: 2018-03-13T21:27:09+00:00
guid: http://www.peoplesoftmods.com/?p=1348
permalink: /psadmin/acm-plugin-configuration-properties/
categories:
  - PeopleSoft Administration
---
I am finally starting to get up to speed with Automated Configuration Management (ACM) Plugins.  ACM is something that the guys over at <a href="http://psadmin.io/tag/ACM/" target="_blank">psadmin.io have been talking about</a> for some time now and I think this is a great new PeopleTools functionality.  I have experienced an unfortanute limitation around the allowed character length of the input configuration properties for the some of the ACM plugins that I am currently working on. It turns out that the input configuration properties for ACM plugins are limited to a measly 254 characters.  This is a problem for plugins that require lengthy configuration properties.  For the plugins that I am creating, I wanted a way to easily create and mange plugin configuration properties without having any character length constraints.

<!--more-->

My hack around this limitation is to use HTML objects created in the Branding Framework to house my lengthy input configuration values for the plugins.  This allows me to modify and maintain the configuration values in the PIA (Branding Objects page) as well as not be limited in the character length of the data.

[<img class="alignnone size-full wp-image-1349" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data.png" alt="Config Data" width="1028" height="663" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data.png 1028w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data-300x193.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data-768x495.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data-1024x660.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data-589x380.png 589w" sizes="(max-width: 1028px) 100vw, 1028px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Data.png)

Then on the page to set the plugin configuration properties, I simply reference the HTML object name that houses the lengthy configuration data.

[<img class="alignnone size-full wp-image-1350" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Property.png" alt="Config Property" width="890" height="481" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Property.png 890w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Property-300x162.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Property-768x415.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Property-703x380.png 703w" sizes="(max-width: 890px) 100vw, 890px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/03/Config_Property.png)

Since we cannot use the GetHtmlText function in an Application Engine program, I had to do another hack to read in the content from the HTML object in the plugin’s configureEnvironment method.  Fortunately, there is a delivered API that allows us to read in the HTML object content data as a string.  I find this API easier to use versus having to query the underlying tables in the DB.

This is how I get the config data from the HTML object in the plugin&#8217;s configureEnvironment method:

<pre><span style="color: #008000;">/* Get the HTML object name */</span>
<span style="color: #0000ff;">Local</span> string &sHtmlObjName = &variables.get(<span style="color: #ff0000;">"env.certobj"</span>);

<span style="color: #008000;">/* Instantiate HTMLDefn object and attempt to open it */</span>
<span style="color: #0000ff;">Local</span> PT_METADATA:MetaDataAPI:HTMLDefn &oHtml = <span style="color: #0000ff;">create</span> PT_METADATA:MetaDataAPI:HTMLDefn(&sHtmlObjName);
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bRet = &oHtml.OpenReadOnly();

<span style="color: #008000;">/* Error if the object doesn't exist */</span>
<span style="color: #0000ff;">If Not</span> (&bRet) <span style="color: #0000ff;">Then</span>
   <span style="color: #0000ff;">throw CreateException</span>(0, 0, <span style="color: #ff0000;">"%1 HTML Object does not exist"</span>, &sHtmlObjName);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #008000;">/* Get the config data defined in the HTML object */</span>
<span style="color: #0000ff;">Local string</span> &sConfigData = &oHtml.ContentData;</pre>

* * *

This technique could potentially work with storing and fetching lengthy configuration properties for any ACM Plugin.  While this method is not as clean as creating a record, page, component, etc. to manage lengthy configuration variables, I think this technique can be a viable option in some cases.