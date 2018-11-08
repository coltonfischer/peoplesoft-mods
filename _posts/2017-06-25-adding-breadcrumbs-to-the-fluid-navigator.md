---
id: 1038
title: Adding Breadcrumbs to the Fluid Navigator
date: 2017-06-25T20:17:35+00:00
guid: http://www.peoplesoftmods.com/?p=1038
permalink: /emf/adding-breadcrumbs-to-the-fluid-navigator/
categories:
  - Event Mapping
  - Tips and Tricks
---
The Fluid Navigator is the new navigation technique that most closely resembles to old Classic style drop down menu navigation.  If you have not yet had the chance to convert your entire menu structure to use more modern navigation techniques, then you are stuck having to rely on the Fluid Navigator to get you where you need to be in some cases.  One major limitation of the Fluid Navigator is that is does not show you breadcrumbs when drilling down into the menu structure.  Not having breadcrumbs displayed makes it much harder to quickly jump around the menu.  In this post, I will demonstrate how you can add breadcrumbs to your Fluid Navigator.

<!--more-->

This solution will be achieved using the Related Content Framework Event Mapping functionality.  We will be mapping two separate application classes to a couple of CREFs in the application.  One application class will be mapped to the component level post build event of the Fluid NavBar.

[<img class="alignnone size-full wp-image-1039" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar.png" alt="Fluid NavBar" width="1135" height="611" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar.png 1135w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar-300x161.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar-768x413.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar-1024x551.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar-706x380.png 706w" sizes="(max-width: 1135px) 100vw, 1135px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-NavBar.png)

And the other application class will be mapped to the component level post build event of the Fluid Navigator.

[<img class="alignnone size-full wp-image-1040" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator.png" alt="Fluid Navigator" width="1125" height="618" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator.png 1125w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator-300x165.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator-768x422.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator-1024x563.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator-692x380.png 692w" sizes="(max-width: 1125px) 100vw, 1125px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Fluid-Navigator.png)

I have included all of the applicable objects for this solution in an App Designer project.

<span style="text-decoration: underline;"><strong><a href="http://peoplesoftmods.com/Development/PSM_BREADCRUMB_NAV.zip">CLICK HERE</a></strong></span> to download the project.

**Project Overview:**

Once you import this project into App Designer, you can open up the PSM\_BREADCRUMB\_NAV Application Package and take a peek at the two classes that will be getting mapped with Event Mapping.  The first class is named DisplayBreadcrumbs.  This class is used to keep track of the portal registry folder objects that the user drills into when navigating.  The folder names serve as the breadcrumbs and they will get displayed at the top of the Fluid Navigator as the user navigates.  When a user clicks on the breadcrumb, the Navigator content area will display the folder’s contents.

<pre><span style="color: #0000ff;">import </span>PT_RCF:ServiceInterface;
<span style="color: #0000ff;">import </span>PTNUI:NavBarContentArea:NBContentCollection;
<span style="color: #0000ff;">import </span>PTNUI:NavBarContentArea:NBContent;

<span style="color: #0000ff;">class</span> DisplayBreadcrumbs <span style="color: #0000ff;">implements</span> PT_RCF:ServiceInterface
 <span style="color: #0000ff;">method</span> execute();
<span style="color: #0000ff;">end-class</span>;

<span style="color: #008000;">/* This is a reference to the current nav bar content collection */</span>
<span style="color: #0000ff;">Component</span> PTNUI:NavBarContentArea:NBContentCollection &ptnui_CurContentColl;

<span style="color: #008000;">/* Use this function to get a reference to the appropriate portal */</span>
<span style="color: #0000ff;">Declare Function</span> PortalOpen <span style="color: #0000ff;">PeopleCode</span> FUNCLIB_PORTAL.PORTAL_GEN_FUNC FieldFormula;

<span style="color: #008000;">/* Use this function to set what shows up in the content (menu) area */</span>
<span style="color: #0000ff;">Declare Function</span> SetContentArea <span style="color: #0000ff;">PeopleCode</span> PTNUI_DOCK_REC.PTNUI_NB_ACTION FieldFormula;

<span style="color: #0000ff;">Declare Function</span> GetLocalNodeContentURI <span style="color: #0000ff;">PeopleCode</span> PTNUI_NB_WRK.FUNCLIB FieldFormula;

<span style="color: #0000ff;">method</span> execute
 <span style="color: #008000;">/+ Extends/implements PT_RCF:ServiceInterface.execute +/</span>
 
<span style="color: #008000;"> /* If the nav bar content collection is blank then return */</span>
 <span style="color: #0000ff;">If None</span>(&ptnui_CurContentColl) <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">Return</span>;
 <span style="color: #0000ff;">End-If</span>;
 
<span style="color: #008000;"> /* The FLDR parameter will be the folder that the user clicked on */</span>
 <span style="color: #0000ff;">Local string</span> &sFolder = <span style="color: #0000ff;">%Request</span>.GetParameter(<span style="color: #ff0000;">"FLDR"</span>);
 
 <span style="color: #0000ff;">Local string</span> &standalone = <span style="color: #0000ff;">%Request</span>.GetParameter(<span style="color: #ff0000;">"sa"</span>);
 
 <span style="color: #008000;">/* If no folder was passed in, then the user must be a the root */</span>
 <span style="color: #0000ff;">If None</span>(&sFolder) <span style="color: #0000ff;">Then</span>
 &sFolder = <span style="color: #ff0000;">"PORTAL_ROOT_OBJECT"</span>;
 <span style="color: #0000ff;">End-If</span>;
 
 <span style="color: #008000;">/* Get a reference to the current portal */</span>
 <span style="color: #0000ff;">Local ApiObject</span> &portal = PortalOpen();
 
<span style="color: #008000;"> /* Get a reference to the current portal registry folder object */</span>
 <span style="color: #0000ff;">Local ApiObject</span> &sParentFolder = &portal.FindFolderByName(&sFolder);
 
 <span style="color: #0000ff;">Local integer</span> &iBreadcrumbCount = 0;
 
 <span style="color: #008000;">/* While the parent folder exists */</span>
 <span style="color: #0000ff;">While All</span>(&sParentFolder)
 
 <span style="color: #008000;">/* Generate the URL to invoke when the breadcrumb is clicked on */</span>
 <span style="color: #0000ff;">Local string</span> &sUrl = GetLocalNodeContentURI() | <span style="color: #ff0000;">"/c/NUI_FRAMEWORK.PTNUI_MENU_COMP.GBL?sa="</span> | &standalone | <span style="color: #ff0000;">"&FLDR="</span> | &sParentFolder.name;
 <span style="color: #0000ff;">If</span> &standalone &lt;&gt; <span style="color: #ff0000;">"y"</span> <span style="color: #0000ff;">Then</span>
 &sUrl = <span style="color: #ff0000;">"javascript:PTNavBar.OpenInContentArea('"</span> | &sUrl | <span style="color: #ff0000;">"&ICDoModal=1&ICGrouplet=1', '"</span> | &sParentFolder.name | <span style="color: #ff0000;">"');"</span>;
 <span style="color: #0000ff;">End-If</span>;
 
 <span style="color: #008000;">/* Change the label for the Root folder */</span>
 <span style="color: #0000ff;">If</span> (&sParentFolder.name = <span style="color: #ff0000;">"PORTAL_ROOT_OBJECT"</span>) <span style="color: #0000ff;">Then</span>
 &sParentFolder.label = <span style="color: #ff0000;">"Main Menu"</span>;
 <span style="color: #0000ff;">End-If</span>;
 
 <span style="color: #008000;">/* Create NBContent object to represent the breadcrumb in the nav bar */</span>
 <span style="color: #0000ff;">Local</span> PTNUI:NavBarContentArea:NBContent &oParentBreadCrumb = <span style="color: #0000ff;">create</span> PTNUI:NavBarContentArea:NBContent(&sParentFolder.name, &sParentFolder.label, &sUrl);
 
 <span style="color: #008000;">/* Set a custom style to differentiate the breadcrumbs from the normal nav bar content items */</span>
 <span style="color: #0000ff;">If</span> (&iBreadcrumbCount &gt; 0) <span style="color: #0000ff;">Then</span>
 &oParentBreadCrumb.style = <span style="color: #ff0000;">"psm_breadcrumb"</span>; <span style="color: #008000;">/* Parent breadcrumb */</span>
 <span style="color: #0000ff;">Else</span>
 &oParentBreadCrumb.style = <span style="color: #ff0000;">"psm_breadcrumb_selected"</span>; <span style="color: #008000;">/* Current breadcrumb */</span>
 <span style="color: #0000ff;">End-If</span>;
 
 <span style="color: #008000;">/* Set the breadcrumb to show at the top of the nav bar content collection */</span>
 &ptnui_CurContentColl.InsertItemAtStart(&oParentBreadCrumb);
 
 <span style="color: #008000;">/* Reference the current folder's parent */</span>
 &sParentFolder = &portal.FindFolderByName(&sParentFolder.ParentName);
 
 &iBreadcrumbCount = &iBreadcrumbCount + 1;
 
 <span style="color: #0000ff;">End-While</span>;
 
 <span style="color: #008000;">/* Set the content area equal to the modified nav bar content collection */</span>
 SetContentArea(&portal, &ptnui_CurContentColl);
 
 <span style="color: #008000;">/* Close the PortalRegistry object */</span>
 &portal.close();
 
<span style="color: #0000ff;">end-method</span>;</pre>

The second class is named StyleBreadcrumbs.  This class is just used to inject custom styles and scripts when the NavBar is clicked by making use of the AddStyleSheet and AddJavaScript functions.  The main style sheet (PSM_BREADCRUMB) will be used to apply a special style to the breadcrumbs so that they cosmetically differ from non-breadcrumb items in the Fluid Navigator.

<pre><span style="color: #0000ff;">import</span> PT_RCF:ServiceInterface;

<span style="color: #0000ff;">class</span> StyleBreadcrumbs <span style="color: #0000ff;">implements</span> PT_RCF:ServiceInterface
 <span style="color: #0000ff;">method</span> execute();
<span style="color: #0000ff;">end-class</span>;

<span style="color: #0000ff;">method</span> execute
 <span style="color: #008000;">/+ Extends/implements PT_RCF:ServiceInterface.execute +/</span>
 
 <span style="color: #008000;">/* Hide the delivered Navigator header */</span>
 <span style="color: #0000ff;">AddStyleSheet</span>(<span style="color: #0000ff;">StyleSheet</span>.PSM_HIDE_NAVIGATOR_HEADER);
 
 <span style="color: #008000;">/* Override the size of the Navigator menu items */</span>
 <span style="color: #0000ff;">AddStyleSheet</span>(<span style="color: #0000ff;">StyleSheet</span>.PSM_SMALL_NAVIGATOR_ITEMS);
 
 <span style="color: #008000;">/* Apply a custom style to the breadcrumbs in the Navigator */</span>
 <span style="color: #0000ff;">AddStyleSheet</span>(<span style="color: #0000ff;">StyleSheet</span>.PSM_BREADCRUMB);
 
 <span style="color: #008000;">/* Automatically open the Navigator when the NavBar is clicked */</span>
 <span style="color: #0000ff;">AddJavaScript</span>(<span style="color: #0000ff;">HTML</span>.PSM_OPEN_NAVIGATOR);
 
<span style="color: #0000ff;">end-method</span>;</pre>

The included PSM_BREADCRUMB style sheet in the project applies a rather basic styling to the breadcrumbs.  I am sure there is a way to make the breadcrumbs more visually appealing, but I do not have an eye for design.  I am certainly open to enhancement suggestions on this!

**Project Configuration:**

Now that we have a basic understanding of the technical details of this project, we are now ready to perform the Event Mapping configuration for this solution.  You will need to navigate to the Event Mapping tab of the Manage Related Content Service page (Main Menu > PeopleTools > Portal > Related Content Service > Manage Related Content Service) and select the &#8220;Map the event of the Application pages” link.  We are interested in performing mappings to the Navigator CREF and the NavBar CREF.

[<img class="alignnone size-full wp-image-1041" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/CREFs.png" alt="Content References" width="721" height="924" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/CREFs.png 721w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/CREFs-234x300.png 234w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/CREFs-297x380.png 297w" sizes="(max-width: 721px) 100vw, 721px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/CREFs.png)

You are going to need to map the PSM\_BREADCRUMB\_DISPLAY Service to the post build/post processing of the Navigator CREF.

[<img class="alignnone size-full wp-image-1042" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Navigator-Event.png" alt="Navigator Event" width="899" height="388" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Navigator-Event.png 899w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Navigator-Event-300x129.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Navigator-Event-768x331.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Navigator-Event-880x380.png 880w" sizes="(max-width: 899px) 100vw, 899px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Navigator-Event.png)

You will need to map the PSM\_BREADCRUMB\_STYLE Service to the post build/post processing of the NavBar CREF.

[<img class="alignnone size-full wp-image-1043" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/NavBar-Event.png" alt="NavBar Event" width="901" height="386" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/NavBar-Event.png 901w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/NavBar-Event-300x129.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/NavBar-Event-768x329.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/NavBar-Event-887x380.png 887w" sizes="(max-width: 901px) 100vw, 901px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/NavBar-Event.png)

After successfully performing and saving the mappings, you should be all set.  All you need to do is log out and log back in to see the changes.

Here is an example of drilling down into the &#8220;Permissions & Roles&#8221; folder in the Fluid Navigator with this bolt on configured:

[<img class="alignnone size-full wp-image-1044" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles.png" alt="Permissions and Roles" width="1120" height="815" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles.png 1120w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles-300x218.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles-768x559.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles-1024x745.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles-522x380.png 522w" sizes="(max-width: 1120px) 100vw, 1120px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/Permissions-and-Roles.png)

In this example, clicking on the PeoleTools folder will bring up the PeopleTools folder contents in the Fluid Navigator content area like this:

[<img class="alignnone size-full wp-image-1045" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools.png" alt="PeopleTools" width="1123" height="810" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools.png 1123w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools-300x216.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools-768x554.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools-1024x739.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools-527x380.png 527w" sizes="(max-width: 1123px) 100vw, 1123px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/PeopleTools.png)

**Additional Navigator Styling:**

You probably noticed in the picture above that the menu items are smaller than they are as delivered.   This is because a custom style sheet (PSM\_SMALL\_NAVIGATOR_ITEMS in the project) was applied in the event mapped code to override the size of the menu items.  I find this style particularly helpful on large form factor devices.  The delivered large sizing of the menu items works well for phones and tablets since the user has less clicking precision.  However, for devices with a lot more screen real estate, I find the large menu items counterproductive. If you are not interested in applying this style to all form factors, then you could always evaluate the user’s form factor in the event mapped code to conditionally apply the custom style.

Another subtle difference is the picture above is the absence of the Navigator header that contains the buttons to go to previous directories.  I found the Navigator header to be useless with the breadcrumbs present.  This is because we can now simply click on the breadcrumb (folder name) to go back to previous directories.  The style sheet that makes this occur is included in the project as well and is named PSM\_HIDE\_NAVIGATOR_HEADER. Similar to the other custom style to adjust the menu item sizing, you have complete control in how you want to apply the style in the event mapped code.

If you decide to not apply the custom styles to alter the Navigator and are only interested in the breadcrumbs, then here is an example of what this would look like:

[<img class="alignnone size-full wp-image-1046" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling.png" alt="No Custom Nav Styling" width="1123" height="1010" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling.png 1123w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling-300x270.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling-768x691.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling-1024x921.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling-423x380.png 423w" sizes="(max-width: 1123px) 100vw, 1123px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/06/No-Custom-Nav-Styling.png)

* * *

While adding breadcrumbs to the Fluid Navigator will hopefully make using it easier, don’t use this as a reason to not explore other navigation techniques that may allow for an even better navigation experience.  There are many ways to perform navigation in a Fluid environment by implementing and using things like Homepages, Tiles, Navigation Collections, Search, etc.  I believe that the proper implementation of these more modern navigation techniques will provide for the best possible overall user experience.