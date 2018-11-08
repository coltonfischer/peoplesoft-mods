---
id: 907
title: Using PT_METADATA to Create Objects
date: 2017-04-02T14:42:32+00:00
guid: http://www.peoplesoftmods.com/?p=907
permalink: /tips-and-tricks/using-pt_metadata-to-create-objects/
categories:
  - Tips and Tricks
---
I was recently poking around the delivered code behind the Branding Framework pages. These are the pages that allow us to create HTML, JavaScript, Style Sheet, and Image definitions within the PIA. These pages are interesting because they prove that app designer is not needed to create these types of objects. It turns out that there is a nice delivered library that is used in the code behind these pages to create these objects. The library is nicely bundled in an application package named PT_METADATA. This package contains all of the necessary classes to create HTML, JavaScript, Style Sheet, and Image definitions. In this post I will share some code samples of how this library can be used to create HTML and Style Sheet objects with PeopleCode.

<!--more-->

For demonstration purposes, I created a page that functions similar to the delivered Branding Framework pages. This page will have a few input fields to specify the needed attributes to create an HTML or Style Sheet object definition.

[<img class="alignnone size-full wp-image-908" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Populated_Fields.png" alt="Populated_Fields" width="652" height="471" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Populated_Fields.png 652w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Populated_Fields-300x217.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Populated_Fields-526x380.png 526w" sizes="(max-width: 652px) 100vw, 652px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Populated_Fields.png)

In this example, I am creating an HTML object. There are only three required fields for HTML object definitions and they are the object name, object description, and the content the object should store. After populating the values and clicking the OK button, a small piece of PeopleCode fires.

[<img class="alignnone size-full wp-image-909" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Create_HTML.png" alt="Create_HTML" width="579" height="380" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Create_HTML.png 579w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Create_HTML-300x197.png 300w" sizes="(max-width: 579px) 100vw, 579px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Create_HTML.png)

<pre>/* Create HTML Object */
import PT_METADATA:MetaDataAPI:HTMLDefn;

Local string &sHTMLObjName = PSM_WRK0.CONTNAME.Value;
Local string &sHTMLObjContent = PSM_WRK0.CONTDATA.Value;
Local string &sHTMLObjDescr = PSM_WRK0.DESCR.Value;

Local PT_METADATA:MetaDataAPI:HTMLDefn &oHTMLMetaData;
&oHTMLMetaData = create PT_METADATA:MetaDataAPI:HTMLDefn(&sHTMLObjName);

If (&oHTMLMetaData.Open()) Then
 &oHTMLMetaData.ContentData = &sHTMLObjContent;
 &oHTMLMetaData.Description = &sHTMLObjDescr;
 If (&oHTMLMetaData.Save()) Then
 MessageBox(0, "", 0, 0, "Saved");
 Else
 MessageBox(0, "", 0, 0, "Save failed");
 End-If;
Else
 Error MsgGetText(0, 0, "Open failed");
End-If;</pre>

This code is using the HTMLDefn class to create and store all of the meta-data for the HTML object definition. This is essentially the same code that fires on the Branding Objects page when you create a new object. After the code fires, the new object can be seen in App Designer.

[<img class="alignnone size-full wp-image-910" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Object.png" alt="New_Object" width="666" height="499" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Object.png 666w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Object-300x225.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Object-507x380.png 507w" sizes="(max-width: 666px) 100vw, 666px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Object.png)

It is worth noting that this same code can be used to edit an existing HTML definition. So in this example, if the PSM\_MY\_HTML object already existed, then this code would’ve overwritten the object’s description and content with the values that were specified in the input fields.

Style Sheets can be created/updated in the same fashion that the HTML objects can. There is a separate class named StylesheetDefn that is used for Style Sheet objects.  Below is sample usage of this class.  As you can see, this class operates similar to the HTMLDefn class.

[<img class="alignnone size-full wp-image-911" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Create_Style.png" alt="Create_Style" width="519" height="397" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Create_Style.png 519w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Create_Style-300x229.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Create_Style-497x380.png 497w" sizes="(max-width: 519px) 100vw, 519px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Create_Style.png)

<pre>/* Create Style Object */
import PT_METADATA:StylesheetDefn;

Local PT_METADATA:StylesheetDefn &oSSMetaData;

Local string &sSSObjName = PSM_WRK0.CONTNAME.Value;
Local string &sSSObjContent = PSM_WRK0.CONTDATA.Value;
Local string &sSSObjDescr = PSM_WRK0.DESCR.Value;

&oSSMetaData = create PT_METADATA:StylesheetDefn(&sSSObjName);

If (&oSSMetaData.Open()) Then
 &oSSMetaData.ExtStyleSheetStr = &sSSObjContent;
 &oSSMetaData.Descr = &sSSObjDescr;
 If (&oSSMetaData.Save()) Then
 MessageBox(0, "", 0, 0, "Saved");
 Else
 MessageBox(0, "", 0, 0, "Save failed");
 End-If;
Else
 Error MsgGetText(0, 0, "Open failed");
End-If;</pre>

Other than providing some exposure to an under documented PeopleTools gem, my usage of the PT_METADATA app package to create objects in this example is not very useful as it mimics delivered functionality. However, I think there can be practical usages of this library in different contexts that can allow for enhancing the process of object/data management.

In my next post, I will provide an [example of using this library to create image objects](http://www.peoplesoftmods.com/psadmin/pt_metadata-and-web-server-cache/). While doing this, I will provide a technique of how this library can provide an avenue to communicate data to be cached on the web server.