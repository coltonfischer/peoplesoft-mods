---
id: 894
title: Classic UI for Administrative Users
date: 2017-04-02T12:15:32+00:00
guid: http://www.peoplesoftmods.com/?p=894
permalink: /emf/classic-ui-for-administrative-users/
categories:
  - Event Mapping
  - Tips and Tricks
---
A common desire among organizations that adopt the Fluid user interface (UI) is the ability to keep the Classic UI for administrative users. The loudest argument backing the need for the Classic UI for admins is that the Fluid UI does not have breadcrumbs. The response to this argument is that the adoption of Fluid is more than just mobile-enabling the application, but it also entails leveraging the new Fluid navigation paradigm which means using homepages, tiles, navigation collections, search, etc., to give users an avenue to perform the transactions that they need to perform.  A proper adoption of the Fluid UI means leveraging these tools to “create your own navigation” for not only self-service users, but admin users as well.  While I don’t necessarily have a dog in this fight, I would like to provide a proof-of-concept example on how one can go about keeping administrative users Classic in a Fluid environment.

<!--more-->

This solution is going to involve a couple of pieces: Role-Based Branding, and Event Mapping. The Role-Based Branding is going to be used to conditionally assign a Classic branding theme for users based on their PeopleSoft roles. The Event Mapping piece is to redirect admin users from the Fluid homepage to the Classic homepage. To get a rundown of each of these functionalities, then I suggest reading Sasank’s <a href="https://pe0ples0ft.blogspot.com/2016/08/peopletools-branding-role-based-themes.html" target="_blank">Role Based Theme Assignment/Override post</a> as well as his <a href="https://pe0ples0ft.blogspot.com/2016/10/emf-hello-world-and-quirks.html" target="_blank">Event Mapping Framework Introduction post</a>.

First, we can tackle the Event Mapping piece of this solution. This will entail writing a simple piece of Application Class PeopleCode to redirect admin users to the Classic homepage.

[<img class="alignnone size-full wp-image-895" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code.png" alt="Redirect_Code" width="1113" height="422" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code.png 1113w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code-300x114.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code-768x291.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code-1024x388.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code-820x312.png 820w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code-1002x380.png 1002w" sizes="(max-width: 1113px) 100vw, 1113px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/1Redirect_Code.png)

<pre>import PT_RCF:ServiceInterface;

class ClassicHomeAdmin implements PT_RCF:ServiceInterface
 method execute();
end-class;

method execute
 /+ Extends/implements PT_RCF:ServiceInterface.execute +/
 /* Take admin users to classic home */
 If IsUserInRole("PAPP_SYSTEM_ADMIN") Then
 Local string &cUrl = GenerateScriptContentURL(%Portal, %Node, Record.WEBLIB_PTBR, Field.ISCRIPT1, "FieldFormula", "IScript_StartPage");
 %Response.RedirectURL(&cUrl | "?HPTYPE=C");
 End-If;
end-method;</pre>

This code checks if the user has an admin-type role using the isUserInRole function and then redirects the user to the start page IScript. The URL parameter HPTYPE with the value of “C” will tell the start page IScript to load the Classic homepage for the user. This code will need to be mapped to the Fluid Homepage component (PT\_LANDINGPAGE) using Event Mapping. Note: I am using the PAPP\_SYSTEM_ADMIN role to signify an admin user.  You can use any role name(s) that you want to signify admin users in your system. Be sure to keep track of the role names that you are using here because you will need to reference them later.

To use Event Mapping to map the Application Class PeopleCode, then a Related Content Service definition must be created. Head over to the Define Related Content Service page and add a new Service by providing an arbitrary Service ID.

[<img class="alignnone size-full wp-image-896" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Related_Content.png" alt="New_Related_Content" width="827" height="433" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Related_Content.png 827w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Related_Content-300x157.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Related_Content-768x402.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Related_Content-726x380.png 726w" sizes="(max-width: 827px) 100vw, 827px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/2New_Related_Content.png)

Populate the required fields and be sure to select Application Class for the URL Type field and input the reference to the Package, Path, and Class Name to the Application Class PeopleCode that was written in the previous step.

[<img class="alignnone size-full wp-image-897" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Related_Content_Defn.png" alt="Related_Content_Defn" width="930" height="825" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Related_Content_Defn.png 930w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Related_Content_Defn-300x266.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Related_Content_Defn-768x681.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Related_Content_Defn-428x380.png 428w" sizes="(max-width: 930px) 100vw, 930px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/3Related_Content_Defn.png)

Now navigate to the Manage Related Content Service page, click on the Event Mapping tab, and select the “Map the event of the Application pages” link.

[<img class="alignnone size-full wp-image-898" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Event_Mapping.png" alt="Event_Mapping" width="839" height="331" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Event_Mapping.png 839w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Event_Mapping-300x118.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Event_Mapping-768x303.png 768w" sizes="(max-width: 839px) 100vw, 839px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/4Event_Mapping.png)

This will bring up a list of the Content References for the Portal. Be sure to check the “include hidden Crefs” checkbox and drill down to the &#8220;Fluid Home&#8221; Content Reference.  This should be found under: Fluid Structure Content > Fluid Pages > PeopleSoft Applications > Fluid Home.

[<img class="alignnone size-full wp-image-899" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/5Fluid_Home_CREF.png" alt="Fluid_Home_CREF" width="843" height="668" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/5Fluid_Home_CREF.png 843w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/5Fluid_Home_CREF-300x238.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/5Fluid_Home_CREF-768x609.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/5Fluid_Home_CREF-480x380.png 480w" sizes="(max-width: 843px) 100vw, 843px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/5Fluid_Home_CREF.png)

Clicking the Fluid Home link will bring up the page to map a Related Content Service to the component level events of the Fluid Homepage (PT_LANDINGPAGE component).  We are going to want to select Pre Build for the Event Name, the newly created Service ID for the Service ID, and Pre Process for the Processing Sequence.  Save the Component.

[<img class="alignnone size-full wp-image-900" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Map_Event.png" alt="Map_Event" width="917" height="697" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Map_Event.png 917w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Map_Event-300x228.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Map_Event-768x584.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Map_Event-500x380.png 500w" sizes="(max-width: 917px) 100vw, 917px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/6Map_Event.png)

At this point if an admin user (a user with the PAPP\_SYSTEM\_ADMIN role) logs in or clicks the home button, then they will be taken to the Classic homepage.

[<img class="alignnone size-full wp-image-901" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home.png" alt="Classic_Home" width="1982" height="842" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home.png 1982w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home-300x127.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home-768x326.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home-1024x435.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home-894x380.png 894w" sizes="(max-width: 1982px) 100vw, 1982px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/7Classic_Home.png)

However, the admin user still does not have access to the Classic style theme that has the drop-down, breadcrumb navigation. We can use the User Attribute Based Theme Assignments in the Branding Framework to enable a Classic theme for admin users.

Head over to the Assign Themes page to assign a theme for the Portal that you are logged in on. On this page, there is a section where we can override the default theme that is applied to the Portal for a given role-based audience.

[<img class="alignnone size-full wp-image-902" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme.png" alt="Classic_Theme" width="1679" height="778" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme.png 1679w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme-300x139.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme-768x356.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme-1024x474.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme-820x380.png 820w" sizes="(max-width: 1679px) 100vw, 1679px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/8Classic_Theme.png)

Since I want a Classic theme to show for admin users, I will specify a theme assignment for the role name PAPP\_SYSTEM\_ADMIN.   You will want to specify the same role name(s) that you are using to identify admin users in the redirection PeopleCode. For this assignment, I specified the Classic style theme named DEFAULT\_THEME\_TANGERINE, but you can specify any Classic theme that you want.  You will also want to ensure that the value you specify for the Priority # field for the assignment is set so that it is the highest priority, unless you want other theme assignment to take precedence.

[<img class="alignnone size-full wp-image-905" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/Theme_Assignment.png" alt="Theme_Assignment" width="1075" height="343" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/Theme_Assignment.png 1075w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/Theme_Assignment-300x96.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/Theme_Assignment-768x245.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/Theme_Assignment-1024x327.png 1024w" sizes="(max-width: 1075px) 100vw, 1075px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/Theme_Assignment.png)

A browser cache clear and web server restart/cache clear might be needed for these changes to work. Now when an admin logs in or clicks the home link, they are presented with the Classic homepage with the drop-down, breadcrumb navigation. The breadcrumb navigation header will stay with the admin user as they navigate around the system.

[<img class="alignnone size-full wp-image-903" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home.png" alt="Classic_Theme_Home" width="1417" height="768" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home.png 1417w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home-300x163.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home-768x416.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home-1024x555.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home-701x380.png 701w" sizes="(max-width: 1417px) 100vw, 1417px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/9Classic_Theme_Home.png)

This is all of the setup that is needed to enable admin users to use the Classic UI in a Fluid environment.

It is worth mentioning that the choice of implementing this sort of solution will most likely bear support implications as it goes against Oracle’s intended usage of the application. Organizations that want to adopt Fluid need to first consider how the delivered Fluid navigation techniques can be leveraged to support all of the navigation use cases before resorting to a custom solution.