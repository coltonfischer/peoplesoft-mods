---
id: 1348
title: ACM Plugin Configuration Properties
date: 2018-03-13T21:27:09+00:00
guid: https://www.peoplesoftmods.com/?p=1348
permalink: /psadmin/acm-plugin-configuration-properties/
tags:
  - ACM
  - Metadata API
  - Branding Framework
categories:
  - PeopleSoft Administration
---

I am finally starting to get up to speed with Automated Configuration Management (ACM) Plugins. ACM is something that the guys over at [psadmin.io have been talking about](http://psadmin.io/tag/ACM/) for some time now and I think this is a great new PeopleTools functionality. I have experienced an unfortanute limitation around the allowed character length of the input configuration properties for the some of the ACM plugins that I am currently working on. It turns out that the input configuration properties for ACM plugins are limited to a measly 254 characters. This is a problem for plugins that require lengthy configuration properties. For the plugins that I am creating, I wanted a way to easily create and mange plugin configuration properties without having any character length constraints.

My hack around this limitation is to use HTML objects created in the Branding Framework to house my lengthy input configuration values for the plugins. This allows me to modify and maintain the configuration values in the PIA (Branding Objects page) as well as not be limited in the character length of the data.

[1]: /assets/images/2018/03/Config_Data.png
[![Config Data][1]][1]

Then on the page to set the plugin configuration properties, I simply reference the HTML object name that houses the lengthy configuration data.

[2]: /assets/images/2018/03/Config_Property.png
[![Config Property][2]][2]

Since we cannot use the GetHtmlText function in an Application Engine program, I had to do another hack to read in the content from the HTML object in the plugin’s configureEnvironment method. Fortunately, there is a delivered API that allows us to read in the HTML object content data as a string. I find this API easier to use versus having to query the underlying tables in the DB.

This is how I get the config data from the HTML object in the plugin's configureEnvironment method:

```java
/* Get the HTML object name */
Local string &sHtmlObjName = &variables.get("env.certobj");

/* Instantiate HTMLDefn object and attempt to open it */
Local PT_METADATA:MetaDataAPI:HTMLDefn &oHtml = create PT_METADATA:MetaDataAPI:HTMLDefn(&sHtmlObjName);
Local boolean &bRet = &oHtml.OpenReadOnly();

/* Error if the object doesn't exist */
If Not (&bRet) Then
   throw CreateException(0, 0, "%1 HTML Object does not exist", &sHtmlObjName);
End-If;

/* Get the config data defined in the HTML object */
Local string &sConfigData = &oHtml.ContentData;
```

* * *

This technique could potentially work with storing and fetching lengthy configuration properties for any ACM Plugin. While this method is not as clean as creating a record, page, component, etc. to manage lengthy configuration variables, I think this technique can be a viable option in some cases.