---
id: 1347
title: Contextual Navigator
date: 2019-10-01T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1348
permalink: /ux/contextual-navigator/
tags:
  - Navigation
  - Event Mapping
  - Tips and Tricks
categories:
  - User Experience
---

The PeopleSoft Navigator is a component that provides users a way to navigate the folders and pages within
Portal Registry menu structure.

[0]: /assets/images/2017/06/Fluid-Navigator.png
[![psNavBar_lastFolder Object][0]][0]

The Navigator is undoubtedly painful to use out of the box, however there are some great techniques offered by
the community to enhance it:

* [Auto expanding the Navigator on NavBar click](https://pe0ples0ft.blogspot.com/2016/08/pt-855-flu-drop-down-menu-vs-navigator.html)

* [Adding Breadcrumbs and styling to the Navigator](https://www.peoplesoftmods.com/emf/adding-breadcrumbs-to-the-fluid-navigator/)

* [Speed up the NavBar slide-out animation](https://jonathan.rehm.me/blog/2019/05/speed-up-navbar/)

I think the community-provided enhancements make the Navigator much more useable.  One outstanding issue that
Oracle or the community hasn’t addressed is the Navigator’s lack of contextual awareness in certain
scenarios.  See [this MOSC idea](https://community.oracle.com/ideas/17359) for details.

As mentioned in the idea, the Navigator does not properly manage the current navigation position.  For
example, let’s say you deep link to the User Profiles page and then open the Navigator.  You will be
presented with the Root of the navigation structure instead of the actual
folder (PeopleTools > Security > User Profiles) that the User Profiles Page resides in. This is very
problematic because the user has no context of where they are if they decide to navigate elsewhere using
the Navigator.

To achieve better contextual awareness within the Navigator, we need to enhance the management of
the `psNavBar_lastFolder` object in the session storage of the browser.  This object holds the
Portal Registry Folder name of the last visited folder within the Navigator and it is what the Navigator
uses to know the current navigation position.

[1]: /assets/images/2019/10/psNavBar_lastFolder.png
[![psNavBar_lastFolder Object][1]][1]

The problem is that this object only ever gets updated when you drill down and open a Content
Reference (CREF) using the Navigator.  Direct navigations using deep links, Recent
Places, Favorites, etc. will not update the `psNavBar_lastFolder` object with the name of the parent
folder of the navigated page.

We can fix this by performing an Event Mapping on the NavBar Component Post Build Event.

[2]: /assets/images/2019/10/EventMappingNavBar.png
[![Event Mapping NavBar][2]][2]

[3]: /assets/images/2019/10/ContextualNavEvent.png
[![Contextual Nav Event][3]][3]

There are two
main pieces of logic that will need to be implemented in the Event Mapped code:

1.	Use the %Request object to obtain the current URL of when the NavBar is clicked and use the Portal
Registry API  to find the CREF of the current URL

2.	Call AddOnLoadScript to inject JavaScript to set the `psNavBar_lastFolder` object value to the parent
name of the obtained CREF

This will make it so that the Navigator will not have a stale (inaccurate) context when it is opened
because the context gets refreshed every time the NavBar is clicked.

Here is the Application Class
PeopleCode to be Event Mapped to the NavBar Component to achieve this functionality:

```java
import PT_RCF:ServiceInterface;

class ContextualNavigator implements PT_RCF:ServiceInterface
   method execute();

   method GetLastNavFolder(&psCurrentUrl As string) Returns string;

end-class;

Declare Function PortalOpen PeopleCode FUNCLIB_PORTAL.PORTAL_GEN_FUNC FieldFormula;


method execute
   /+ Extends/implements PT_RCF:ServiceInterface.execute +/

   Local string &sCurrentUrl = %Request.GetHeader("Referer");

   AddOnLoadScript("PTNavBarNavigator.SetLastNavFolder('" | %This.GetLastNavFolder(&sCurrentUrl) | "');");

end-method;


method GetLastNavFolder
   /+ &psCurrentUrl as String +/
   /+ Returns String +/

   Local ApiObject &oPortal = PortalOpen();

   Local ApiObject &oCref = &oPortal.FindCRefByURL(&psCurrentUrl);

   If &oCref = Null Then
      &oPortal.close();
      Return "";
   End-If;

   Local string &sLastNavFolder = &oCref.ParentName;

   &oCref = &oPortal.FindFolderByName(&sLastNavFolder);

   /* Do not use the Folder if it is hidden from Portal Nav */
   While &oCref <> Null

      If Not (&oCref.IsVisible) Then
         &oPortal.close();
         Return "";
      End-If;

      &oCref = &oPortal.FindFolderByName(&oCref.ParentName);

   End-While;

   &oPortal.close();

   Return &sLastNavFolder;

end-method;
```