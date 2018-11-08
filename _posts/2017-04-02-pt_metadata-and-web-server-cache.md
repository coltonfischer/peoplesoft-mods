---
id: 914
title: PT_METADATA and Web Server Cache
date: 2017-04-02T15:09:09+00:00
guid: http://www.peoplesoftmods.com/?p=914
permalink: /psadmin/pt_metadata-and-web-server-cache/
categories:
  - PeopleSoft Administration
  - Tips and Tricks
---
A while back I presented a solution to [manage configuration data on the web server](http://www.peoplesoftmods.com/psadmin/managing-configuration-data-on-the-web-server/). This solution involved a servlet-based cache managing utility that was responsible for the storage and caching of data to the PeopleSoft web server. I have been experimenting with ways that I can get custom data cached to the PeopleSoft web server without being so intrusive to the web-tier. I recently came across [the PT_METADATA app package](http://www.peoplesoftmods.com/tips-and-tricks/using-pt_metadata-to-create-objects/) which can be used to create object types such as HTML, Style Sheet, and Images. These object types are nice because they are PeopleTools-managed and they have the flexibility to store custom data expressed in any form. Another great perk of these object types is that they can be cached to the web server using the %Response class. I am going to demonstrate a technique that I use to get PeopleTools to manage and cache my custom data.

<!--more-->

This solution is going to involve using the PT_METADATA app package to create new image objects with PeopleCode.  For demonstration purposes, I created a custom page to gather the required fields to create an Image object.

[<img class="alignnone size-full wp-image-915" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Required_Fields.png" alt="Required_Fields" width="675" height="480" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Required_Fields.png 675w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Required_Fields-300x213.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Required_Fields-534x380.png 534w" sizes="(max-width: 675px) 100vw, 675px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Required_Fields.png)

The four required fields to create an image object are the object name, object description, object file extension, and the content to store in the object. The object’s file extension is limited to three characters, but can be set to any three characters. This allows us to have non-image-type extensions such as htm, js, css, txt, dat, or log for example. Also, the data that can be stored in the image object can be of any form. We can store HTML, XML, or JSON for example.  Most importantly however, the type of data that an image object can hold is arbitrary. Image objects do not actually have to hold images/image data just like HTML objects don’t have to hold actual HTML. This is why these object types are so great because they are a Tools objects that have the flexibility to store custom data.

As you see in the picture above, I am storing some JSON data in an image object with a “dat” file extension. When I click the OK button, a small piece of PeopleCode fires to create a new image object and cache the object to the web server cache directory.

[<img class="alignnone size-full wp-image-916" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object.png" alt="Create_Image_Object" width="1147" height="532" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object.png 1147w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object-300x139.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object-768x356.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object-1024x475.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object-819x380.png 819w" sizes="(max-width: 1147px) 100vw, 1147px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Create_Image_Object.png)

<pre>/* Create Image Object */
import PT_METADATA:MetaDataAPI:ImageDefn;

Local string &sIMGObjName = PSM_WRK0.CONTNAME.Value;
Local string &sFileExtension = PSM_WRK0.EXTENSION.Value;
Local string &sIMGObjDescr = PSM_WRK0.DESCR.Value;
Local Field &fIMGObjContent = GetLevel0().GetRow(1).GetRecord(Record.PSM_WRK0).CONTDATA;

Local PT_METADATA:MetaDataAPI:ImageDefn &oIMGMetaData;
&oIMGMetaData = create PT_METADATA:MetaDataAPI:ImageDefn(&sIMGObjName);

If (&oIMGMetaData.Open()) Then
 &oIMGMetaData.SetContentData(&sFileExtension, &fIMGObjContent);
 &oIMGMetaData.Description = &sIMGObjDescr;
 If (&oIMGMetaData.Save()) Then
 MessageBox(0, "", 0, 0, "Saved");
 Else
 MessageBox(0, "", 0, 0, "Save failed");
 End-If;
Else
 Error MsgGetText(0, 0, "Open failed");
End-If;

/* Cache the file */
Local string &sIMGName = "IMAGE." | &sIMGObjName;
Local string &sCachedURL = %Response.GetImageURL(@&sIMGName);

/* Only show the non-versioned filename if ENABLENOVERSION is checked on the web profile */
Local string &sCachedURLNoVersion = Substring(&sCachedURL, 1, Find(&sIMGObjName, &sCachedURL) - 1) | &sIMGObjName | "." | &sFileExtension;
PSM_WRK0.HTMLAREA.Value = "&lt;br&gt;&lt;div class=""PSEDITBOXLABEL""&gt;&lt;b&gt;Cached URL: &lt;/b&gt;&lt;a href=""" | &sCachedURL | """target=""_blank""&gt;" | &sCachedURL | "&lt;/a&gt;&lt;/div&gt;";
PSM_WRK0.HTMLAREA.Value = PSM_WRK0.HTMLAREA.Value | "&lt;br&gt;&lt;div class=""PSEDITBOXLABEL""&gt;&lt;b&gt;Cached URL (No Version): &lt;/b&gt;&lt;a href=""" | &sCachedURLNoVersion | """target=""_blank""&gt;" | &sCachedURLNoVersion | "&lt;/a&gt;&lt;/div&gt;";</pre>

The code uses the ImageDefn class to create an image object based on the provided input values. After the image object is created, the GetImageURL method of the %Response class is invoked to get the newly created data stored to the web server’s cache directory. The cached URLs are presented to the user at this point.

[<img class="alignnone size-full wp-image-917" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2Cache_URLs.png" alt="Cache_URLs" width="668" height="506" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2Cache_URLs.png 668w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2Cache_URLs-300x227.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2Cache_URLs-502x380.png 502w" sizes="(max-width: 668px) 100vw, 668px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2Cache_URLs.png)

The GetImageURL method will create two cached URLs for the data file.  The first one is the versioned one and the second is the non-versioned one.  Here is a look at the web server’s cache directory.

[<img class="alignnone size-full wp-image-918" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Cached_Files.png" alt="Cached_Files" width="761" height="374" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Cached_Files.png 761w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Cached_Files-300x147.png 300w" sizes="(max-width: 761px) 100vw, 761px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Cached_Files.png)

The reason that there were two URLs that got generated is due to a setting on the web profile that allows for a non-versioned copy of cached Image and Style objects to be generated in the web server’s cached directory.  This setting can be toggled by checking the checkbox labeled “Copy Image/CSS (no Versioning)”under the caching tab of the web profile.

[<img class="alignnone size-full wp-image-919" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Enable_No_Version.png" alt="Enable_No_Version" width="997" height="455" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Enable_No_Version.png 997w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Enable_No_Version-300x137.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Enable_No_Version-768x350.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Enable_No_Version-833x380.png 833w" sizes="(max-width: 997px) 100vw, 997px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Enable_No_Version.png)

What this setting does is it forces the web server to maintain two separate versions of cached image and css files. This allows for one version of the file to keep a static filename, while the other copy has a version number appended to the end of it. This is a great feature because the non-versioned version of the cached file is always reference-able since its filename stays the same, but its content will always be up to date. This, along with the file extension flexibility is the main reasons that I am using Image objects to house my custom data verses HTML or Style objects. Side Note: The “Copy Image/CSS (no Versioning)” functionality on the web profile did not work for me when I would cache Style objects. This is another reason that I am using Image objects with this solution.

So this is how one can go about generating data on the fly and immediately getting it cached to the web server.  I like this solution because it allows for the storage and caching to be managed completely by PeopleTools. One thing to keep in mind is the sensitivity of data that you are caching. I am using this solution to cache non-sensitive configuration data. I advise to not cache any sensitive data with this solution as the cache directory is available to any authenticated user (or non-authenticated user if the public user is enabled).