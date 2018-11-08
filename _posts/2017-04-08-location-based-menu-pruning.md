---
id: 931
title: Location-Based Menu Pruning
date: 2017-04-08T14:09:56+00:00
guid: http://www.peoplesoftmods.com/?p=931
permalink: /emf/location-based-menu-pruning/
categories:
  - Event Mapping
---
Menu pruning is the process of limiting the items that show up in a menu for a user. This process is desirable in situations where you want to prevent a user from accessing certain content references. An example scenario of this could be that you don’t want to let administrators access Query Viewer when they are not coming in from a trusted location. The fashion in which the administrator’s access is limited in this scenario is the process of location-based security. With location-based security, you can let the location that a user is coming from dictate the type of access that a user has in the application. So if we put these two terms together, we get location-based menu pruning. Location-based menu pruning is the process of limiting the items that show up in the menu for a user based on the user’s location.  In this post, I will discuss how we can perform location-based menu pruning on the PeopleSoft Fluid Navigator.

<!--more-->

* * *

I will be demonstrating various ways that we can alter the Navigator menu that gets displayed to a user at run time. This will be achieved by using Event Mapping to map custom code to the post-processing of the Navigator Component&#8217;s Post Build event. The name of the Component used for the Navigator is PTNUI\_MENU\_COMP. I will be sharing some code samples that will need to be mapped with Event Mapping.  If you are unsure of what Event Mapping is or how to use it, then I recommend reading <a href="https://pe0ples0ft.blogspot.com/2016/10/emf-hello-world-and-quirks.html" target="_blank">this introductory Event Mapping post</a> written by Sasank Vemana.

Before I jump into the menu pruning examples, I want to clarify one really important piece of information. Menu pruning in itself is a user interface enhancement and NOT a security enhancement. Menu pruning is just going to remove the available Navigator (menu) paths that a user has to particular content references. So if you prune out the “Query Viewer” CREF from the menu, then you are just removing the avenue that the user has to that CREF. If the user knows the URL for the “Query Viewer” CREF, then they can simply paste it in their browser and have access to the component. If you are interested in actually securing the CREFs that you are pruning from the menu, then I suggest reading this post on [Enabling Location Based Security in PeopleSoft with Event Mapping](http://www.peoplesoftmods.com/emf/enabling-location-based-security-in-peoplesoft-with-event-mapping/).

With that out of the way, let’s take a look at some location-based menu pruning examples. The code samples provided for each of these examples is heavily commented, so I believe that it can explain itself. However, I will give a high-level overview of what each of these code samples are doing. Note: This code was tested on an Interaction Hub system running 8.55.12 PeopleTools.

**Example of Deleting Menu Items Based on a User&#8217;s Location**

This first example is fairly straight-forward. What is happening here is that we are getting a reference to the current NBContetCollection object from the component buffer. This object holds a collection of all of the menu items that will get displayed to the user in the Navigator. The NBContetCollection class has a DeleteItem method that will delete a menu item (CREF) from the collection, which results in the deleted CREF to not be displayed. In this example, the code first checks if the user is coming from a trusted location before calling the DeleteItem method to delete the desired CREFs.

[<img class="alignnone size-full wp-image-937" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/DeleteExample.png" alt="DeleteExample" width="960" height="798" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/DeleteExample.png 960w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/DeleteExample-300x249.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/DeleteExample-768x638.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/DeleteExample-457x380.png 457w" sizes="(max-width: 960px) 100vw, 960px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/DeleteExample.png)

<pre>import PT_RCF:ServiceInterface;
import PTNUI:NavBarContentArea:NBContentCollection;
import PTNUI:NavBarContentArea:NBContent;

class EMF2 implements PT_RCF:ServiceInterface
 method execute();
end-class;

/* This is a reference to the current nav bar content collection */
Component PTNUI:NavBarContentArea:NBContentCollection &ptnui_CurContentColl;

/* Use this function to get a reference to the appropriate portal */
Declare Function PortalOpen PeopleCode FUNCLIB_PORTAL.PORTAL_GEN_FUNC FieldFormula;

/* Use this function to set what shows up in the content (menu) area */
Declare Function SetContentArea PeopleCode PTNUI_DOCK_REC.PTNUI_NB_ACTION FieldFormula;

/* Location-based menu pruning example. Note: You can prune the menu based on any conditions (doesn't have to be location) */
method execute
 /+ Extends/implements PT_RCF:ServiceInterface.execute +/
 
 /* Check if the user is coming from a trusted location */
 /* TODO: store trusted IP address ranges in setup table to avoid hard-coding IP addresses */
 If (All(&ptnui_CurContentColl) And
 %Request.RemoteAddr &lt;&gt; "192.168.56.10") Then
 
 /* Delete the PeopleTools folder CREF */
 /* TODO: Store CREFs to delete in setup table to avoid hard-coding */
 &ptnui_CurContentColl.DeleteItem("PT_PEOPLETOOLS");
 
 /* Delete the Change My Password CREF */
 &ptnui_CurContentColl.DeleteItem("PT_CHANGE_PASSWORD_GBL");
 
 /* Delete the Query Viewer CREF */
 &ptnui_CurContentColl.DeleteItem("PT_QUERY_VIEWER_GBL");
 
 
 /* Get a reference to the current portal */
 Local ApiObject &portal = PortalOpen();
 
 /* Set the content area equal to the modified nav bar content collection */
 SetContentArea(&portal, &ptnui_CurContentColl);
 
 /* The Close method closes the PortalRegistry object */
 &portal.close();
 
 End-If;
 
end-method;</pre>

&nbsp;

**Example of Changing the Style of Menu Items Based on a User&#8217;s Location**

In this next example, we are altering the style for a given CREF that will be displayed to a user. Specifically, we are overriding the style of the PeopleTools folder by setting it to the “ps\_hidden” style. This style simply hides the element that it is applied to. So the output for this example is actually the same as if we were to delete the item. However, with this example, the PeopleTools folder could be exposed by inspecting the page’s source. Applying the “ps\_hidden” style is not very practical, but I just wanted to demo how one can override the style of a given menu item. A more practical use case could be to inject a custom style using the AddStyleSheet function and then override the menu items style with the custom one.

[<img class="alignnone size-full wp-image-938" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample1.png" alt="AlterStyleExample1" width="1019" height="558" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample1.png 1019w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample1-300x164.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample1-768x421.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample1-694x380.png 694w" sizes="(max-width: 1019px) 100vw, 1019px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample1.png)[<img class="alignnone size-full wp-image-939" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample2.png" alt="AlterStyleExample2" width="1019" height="584" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample2.png 1019w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample2-300x172.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample2-768x440.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample2-663x380.png 663w" sizes="(max-width: 1019px) 100vw, 1019px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterStyleExample2.png)

<pre>import PT_RCF:ServiceInterface;
import PTNUI:NavBarContentArea:NBContentCollection;
import PTNUI:NavBarContentArea:NBContent;

class EMF2 implements PT_RCF:ServiceInterface
 method execute();
end-class;

/* This is a reference to the current nav bar content collection */
Component PTNUI:NavBarContentArea:NBContentCollection &ptnui_CurContentColl;

/* Use this function to get a reference to the appropriate portal */
Declare Function PortalOpen PeopleCode FUNCLIB_PORTAL.PORTAL_GEN_FUNC FieldFormula;

/* Use this function to set what shows up in the content (menu) area */
Declare Function SetContentArea PeopleCode PTNUI_DOCK_REC.PTNUI_NB_ACTION FieldFormula;

/* Location-based menu pruning example. Note: You can prune the menu based on any conditions (doesn't have to be location) */
method execute
 /+ Extends/implements PT_RCF:ServiceInterface.execute +/
 
 /* Check if the user is coming from a trusted location */
 If (All(&ptnui_CurContentColl) And
 %Request.RemoteAddr &lt;&gt; "192.168.56.10") Then
 
 /* Create a Nav Bar Content Collection variable to store the altered menu */
 Local PTNUI:NavBarContentArea:NBContentCollection &ptnui_CustomContentColl = create PTNUI:NavBarContentArea:NBContentCollection();
 
 /* This will hold the current menu item CREF */
 Local PTNUI:NavBarContentArea:NBContent &oCurrentMenuItem;
 
 /* Start off by setting the current menu item CREF to the first menu item CREF in the collection */
 &oCurrentMenuItem = &ptnui_CurContentColl.First();
 
 /* Loop through all of the menu item CREFs in the collection */
 While (All(&oCurrentMenuItem))
 
 /* Apply the the "ps_hidden" style to the PeopleTools folder CREF */
 If (&oCurrentMenuItem.getName() = "PT_PEOPLETOOLS") Then
 
 /* Set the style of the menu item as you please */
 &oCurrentMenuItem.style = "ps_hidden";
 
 End-If;
 
 /* Insert the current menu item CREF into the modified menu content collection */
 &ptnui_CustomContentColl.InsertItem(&oCurrentMenuItem);
 
 /* Set the current menu item CREF to the next menu item CREF in the collection */
 &oCurrentMenuItem = &ptnui_CurContentColl.Next();
 
 End-While;
 
 /* Set the component Nav Bar Content Collection variable to the modified version */
 &ptnui_CurContentColl = &ptnui_CustomContentColl;
 
 /* Get a reference to the current portal */
 Local ApiObject &portal = PortalOpen();
 
 /* Set the content area equal to the modified nav bar content collection */
 SetContentArea(&portal, &ptnui_CurContentColl);
 
 /* The Close method closes the PortalRegistry object */
 &portal.close();
 
 End-If;
 
end-method;</pre>

&nbsp;

**Example of Changing the Action of Menu Items Based on a User&#8217;s Location**

In this last example, we are overriding the action that occurs when a user clicks on a particular CREF. Specifically, what we have done is overridden the location that a user is taken to when the user clicks the “Change My Password” menu item. In this example, the user will be taken to the <http://www.peoplesoftinfo.com> address when they click the “Change My Password” menu item. I believe there can be many practical uses of this functionality.

[<img class="alignnone size-full wp-image-945" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1.png" alt="AlterActionExample1" width="1329" height="558" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1.png 1329w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1-300x126.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1-768x322.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1-1024x430.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1-905x380.png 905w" sizes="(max-width: 1329px) 100vw, 1329px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample1.png)[<img class="alignnone size-full wp-image-946" src="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2.png" alt="AlterActionExample2" width="1326" height="662" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2.png 1326w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2-300x150.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2-768x383.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2-1024x511.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2-761x380.png 761w" sizes="(max-width: 1326px) 100vw, 1326px" />](http://www.peoplesoftmods.com/wp-content/uploads/2017/04/AlterActionExample2.png)

<pre>import PT_RCF:ServiceInterface;
import PTNUI:NavBarContentArea:NBContentCollection;
import PTNUI:NavBarContentArea:NBContent;

class EMF2 implements PT_RCF:ServiceInterface
 method execute();
end-class;

/* This is a reference to the current nav bar content collection */
Component PTNUI:NavBarContentArea:NBContentCollection &ptnui_CurContentColl;

/* Use this function to get a reference to the appropriate portal */
Declare Function PortalOpen PeopleCode FUNCLIB_PORTAL.PORTAL_GEN_FUNC FieldFormula;

/* Use this function to set what shows up in the content (menu) area */
Declare Function SetContentArea PeopleCode PTNUI_DOCK_REC.PTNUI_NB_ACTION FieldFormula;

/* Location-based menu pruning example. Note: You can prune the menu based on any conditions (doesn't have to be location) */
method execute
 /+ Extends/implements PT_RCF:ServiceInterface.execute +/
 
 /* Check if the user is coming from a trusted location */
 If (All(&ptnui_CurContentColl) And
 %Request.RemoteAddr &lt;&gt; "192.168.56.10") Then
 
 /* Create a Nav Bar Content Collection variable to store the altered menu */
 Local PTNUI:NavBarContentArea:NBContentCollection &ptnui_CustomContentColl = create PTNUI:NavBarContentArea:NBContentCollection();
 
 /* This will hold the current menu item CREF */
 Local PTNUI:NavBarContentArea:NBContent &oCurrentMenuItem;
 
 /* Start off by setting the current menu item CREF to the first menu item CREF in the collection */
 &oCurrentMenuItem = &ptnui_CurContentColl.First();
 
 /* Loop through all of the menu item CREFs in the collection */
 While (All(&oCurrentMenuItem))
 
 /* Alter the click action of the Change My Password CREF */
 If (&oCurrentMenuItem.getName() = "PT_CHANGE_PASSWORD_GBL") Then
 
 /* Set the action URL to open the PeopleSoftInfo website */
 Local string &sUrl = "javascript:PTNavBar.OpenInWindow('" | "http://www.peoplesoftinfo.com" | "');";
 
 /* Create a modified version of the current menu item that will open the PeopleSoftInfo website when clicked */
 Local PTNUI:NavBarContentArea:NBContent &oModifiedMenuItem = create PTNUI:NavBarContentArea:NBContent(&oCurrentMenuItem.getName(), &oCurrentMenuItem.getLabel(), &sUrl);
 &oModifiedMenuItem.style = &oCurrentMenuItem.style;
 &oModifiedMenuItem.ariaAttributes = &oCurrentMenuItem.ariaAttributes;
 &oCurrentMenuItem = &oModifiedMenuItem;
 
 End-If;
 
 /* Insert the current menu item CREF into the modified menu content collection */
 &ptnui_CustomContentColl.InsertItem(&oCurrentMenuItem);
 
 /* Set the current menu item CREF to the next menu item CREF in the collection */
 &oCurrentMenuItem = &ptnui_CurContentColl.Next();
 
 End-While;
 
 /* Set the component Nav Bar Content Collection variable to the modified version */
 &ptnui_CurContentColl = &ptnui_CustomContentColl;
 
 /* Get a reference to the current portal */
 Local ApiObject &portal = PortalOpen();
 
 /* Set the content area equal to the modified nav bar content collection */
 SetContentArea(&portal, &ptnui_CurContentColl);
 
 /* The Close method closes the PortalRegistry object */
 &portal.close();
 
 End-If;
 
end-method;</pre>

&nbsp;

* * *

&nbsp;

These are a few ways that we can dynamically change the look and feel of the Navigator. In these examples, the alterations of the menu items were conditioned upon the user’s location. It should be clear that you can alter the menu items based on any condition(s) and not just location.

While I think this solution has potential to be powerful, the effectiveness of it can vary depending on the navigation implementation for a given PeopleSoft system. What I mean by this is that the Navigator menu may not be the only avenue to a given CREF. PeopleSoft delivers many other navigation utilities along with the Navigator. I would like it to be well understood that the solutions that I have demonstrated today is just for altering the behavior of the Navigator, and not the other delivered navigation techniques/utilities.