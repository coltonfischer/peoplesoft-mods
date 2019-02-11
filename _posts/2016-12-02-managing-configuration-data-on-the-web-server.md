---
id: 782
title: Managing Configuration Data on the Web Server
date: 2016-12-02T15:36:43+00:00
author: Colton Fischer
guid: https://www.peoplesoftmods.com/?p=782
permalink: /psadmin/managing-configuration-data-on-the-web-server/
categories:
  - PeopleSoft Administration
  - Servlet Filters
---
Some of the servlet filters that I have previously demonstrated make use of configuration data that originates from the PeopleSoft application.  This configuration data consists of things like which fields to mask and what components to restrict access to.  This data is ultimately what controls how the servlet filters behave.  In this post, I would like to discuss how I go about communicating configuration data from the PeopleSoft application to the web server and the techniques that I use to manage this data.  First I will go over the design, installation, and use of a caching utility that I wrote to manage the configuration data.  Last I will discuss how I integrate my servlet filters and PeopleSoft application with this solution.

<!--more-->

**Cache Manager Design:**

What I have done is written a cache manager servlet.  This servlet is responsible for the storage of the configuration data that I want to be able to access from my servlet filters.  The cache manager servlet stores the data in two different ways:

  1. A text file that resides in a web server directory
  2. A variable that resides in the web server memory

The servlet will first load the configuration data to the text file on the web server.  This action is performed via a push from the PeopleSoft application.  This push is achieved by sending a specially formed HTTP GET request to the cache manager servlet.

[<img class="alignnone size-full wp-image-783" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Data_Source.png" alt="Set_Data_Source" width="661" height="373" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Data_Source.png 661w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Data_Source-300x169.png 300w" sizes="(max-width: 661px) 100vw, 661px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Data_Source.png)

Once the text file has data loaded into it, the servlet can set a variable within memory equal to the data from the text file.  Similar to the above methodology, the action of setting\updating this variable is achieved by a push from the PeopleSoft application.

[<img class="alignnone size-full wp-image-784" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Cache.png" alt="Update_Cache" width="678" height="372" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Cache.png 678w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Cache-300x165.png 300w" sizes="(max-width: 678px) 100vw, 678px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Cache.png)

With the configuration data stored in the web server memory, all of the servlet filters will have an efficient avenue to access the data and behave dynamically.

[<img class="alignnone size-full wp-image-785" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Dynamic_Servlet_Filter.png" alt="Dynamic_Servlet_Filter" width="673" height="354" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Dynamic_Servlet_Filter.png 673w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Dynamic_Servlet_Filter-300x158.png 300w" sizes="(max-width: 673px) 100vw, 673px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Dynamic_Servlet_Filter.png)

The reason for the intermediate storage of the configuration data in the text file is so that there will be a backup of the data for when the memory is flushed on web server shutdown.  Having this data backed up to the file allows for the web server to re-initialize the configuration data in the memory when it starts up (more details on this below).

&nbsp;

**Cache Manager Installation:**

Now I will show how to set up the cache manager servlet in the PeopleSoft web server.  [CLICK HERE](https://www.peoplesoftmods.com/Development/PSM_CACHE_MANAGER_POC.zip) to download the needed files.

Copy the contents of the downloaded PORTAL.war folder into the PORTAL.war folder on your web server.  On your web server, drill down into the newly copied psm folder (PORTAL.war \ps\psm) and make note of the absolute file paths to the MaskRules.txt file and the RedirectRules.txt file.

[<img class="alignnone size-full wp-image-786" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/File_Paths.png" alt="File_Paths" width="890" height="487" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/File_Paths.png 890w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/File_Paths-300x164.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/File_Paths-768x420.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/File_Paths-694x380.png 694w" sizes="(max-width: 890px) 100vw, 890px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/File_Paths.png)

Next, open up the downloaded webxml.txt file and set the DMR parameter value equal to the file path to the MaskRules.txt file. Set the CRR parameter value equal to the file path to the RedirectRules.txt file.  Now copy the contents of the modified webxml.txt file and paste it into the web.xml file located on your web server.  Paste in the text before the psc servlet definition.

[<img class="alignnone size-full wp-image-787" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML.png" alt="Web_XML" width="1191" height="433" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML.png 1191w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML-300x109.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML-768x279.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML-1024x372.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML-1045x380.png 1045w" sizes="(max-width: 1191px) 100vw, 1191px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Web_XML.png)

A web server bounce will be needed at this point.

&nbsp;

**Using the Cache Manager:**

After installing the Cache Manager Servlet on your desired web server, you can access the API details of how to use it by going to http://BaseURL:Port/CacheManager in your web browser.

[<img class="alignnone size-full wp-image-788" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/API_Details.png" alt="API_Details" width="715" height="457" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/API_Details.png 715w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/API_Details-300x192.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/API_Details-595x380.png 595w" sizes="(max-width: 715px) 100vw, 715px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/API_Details.png)

The first action you will want to do is update your data source file for one of the valid data types.  The valid data types are the param-name values that are specified in the web.xml.  In the example that I have provided, there are two valid data types: DMR and CRR.  It is worth mentioning that you can have as few or as many data types that you want.  The point of making these values was to have a clear separation of different types of configuration data that you may want the servlet to maintain.  The data type parameter is merely just a pointer to the data source text files.  Here is an example of how to update the data source file for the data masking rules (DMR):

[<img class="alignnone size-full wp-image-789" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Data_Source.png" alt="Update_Data_Source" width="1044" height="337" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Data_Source.png 1044w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Data_Source-300x97.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Data_Source-768x248.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Data_Source-1024x331.png 1024w" sizes="(max-width: 1044px) 100vw, 1044px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Update_Data_Source.png)

If you get the “Data Source Updated” response from the servlet, then the contents of the text file have been successfully updated.  The content of the MaskRules.txt file now reads “test123”.  You can manually go to the MaskRules.txt file on the web server to see the file’s contents or you can invoke the servlet to retrieve the contents of the file by specifying the DMR data type and the parameter &getDataSource.

[<img class="alignnone size-full wp-image-790" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Data_Source.png" alt="Get_Data_Source" width="1051" height="318" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Data_Source.png 1051w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Data_Source-300x91.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Data_Source-768x232.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Data_Source-1024x310.png 1024w" sizes="(max-width: 1051px) 100vw, 1051px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Data_Source.png)

Now that the data source is loaded with updated content, we need to invoke the servlet to update the cache memory with the content from the data source.  To do this, we need to specify the data type for the cached content that we want to update as well as the parameter &setCache.  Since we want to update the data masking rules, then we will specify DMR for the data type.

[<img class="alignnone size-full wp-image-791" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Cache.png" alt="Set_Cache" width="1044" height="311" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Cache.png 1044w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Cache-300x89.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Cache-768x229.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Cache-1024x305.png 1024w" sizes="(max-width: 1044px) 100vw, 1044px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Set_Cache.png)

If you get the “Cache Updated” response from the servlet, then the contents of the cache have been successfully updated.  The contents of the data masking rules cache should now be equal to the contents of the data masking rules data source.  To check, you can invoke the servlet to return the contents of the cache by specifying the DMR data type and the parameter &getCache.

[<img class="alignnone size-full wp-image-792" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Cache.png" alt="Get_Cache" width="1043" height="315" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Cache.png 1043w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Cache-300x91.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Cache-768x232.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Cache-1024x309.png 1024w" sizes="(max-width: 1043px) 100vw, 1043px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/12/Get_Cache.png)

As I mentioned in the design section, the reason that I use the intermediate data source (.txt file) to store the content is so that the cached content can be re-initialized on web server startup.  This means that if you were to restart your web server at this point, then the “test123” value will automatically be re-loaded into the DMR cache on startup.  This was achievable by writing code that loads the cache with the data from the data source file in the init method of the servlet.  Specifying a 1 for the load-on-startup parameter in the web.xml file is what allows the Cache Manager&#8217;s init method to be invoked on startup.  Without this parameter, the servlet would have to be manually invoked after web server startup to re-load the cache memory.

&nbsp;

**Integrating the Cache Manager:**

Now I will discuss how I reference and make use of the cached content from within my servlet filters and how I go about communicating configuration data from my PeopleSoft application to the Cache Manager Servlet.  Accessing the cached content from within a servlet filter is very simple.  You will first need to import the CacheManager class in your servlet filter:

<pre>Import com.peoplesoft.pt.custom.CacheManager;</pre>

Now inside of the doFilter method of the servlet filter, the Cache Manager’s “getCache” method can be invoked to get the cached contents for a particular data type.  Here is the Java code to get the cached content for the DMR data type:

<pre>String MaskRules = CacheManager.getCache(“DMR”);</pre>

Now the “MaskRules” string will contain the dynamic data masking rules so that the servlet filter can behave off of this information.

I showed above how the data source and cache can be updated from within a web browser, but this needs to be done from within the PeopleSoft application for this solution to be useful.  I make use of Integration Broker to integrate this solution into my PeopleSoft application.  With a properly configured Integration Broker, the process of the updating the data source and cached content is easy with the following PeopleCode:

<pre>&makingRules = “test123”;

&output = %IntBroker.ConnectorRequestUrl(%Request.Scheme | "://" | %Request.ServerName | ":" | %Request.ServerPort | "/CacheManager?dataType=DMR&setCache&data=" | &makingRules);</pre>

This single command will update both the data source and the cached content on the web server.  This allows the servlet filter to immediately start behaving off of the newly updated data without the need of a web server bounce.