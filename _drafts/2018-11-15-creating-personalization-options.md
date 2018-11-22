---
id: 1343
title: Creating Personalization Options
date: 2018-11-14
guid: http://www.peoplesoftmods.com/?p=1343
permalink: /ux/creating-personalization-options/
tags:
  - User Personalizations
  - Branding Framework
categories:
  - User Experience
---

In my previous post, I described how we can edit existing Personalization Options to allow 
for [User Preference-Based Homepage Types](/ux/user-preference-based-homepage-types/).  I am going to expand on this same topic and demonstrate how to create a 
custom Personalization Option that will allow end users to select a Portal Branding Theme on their My Preferences page.  The end goal here is to allow end users 
to select their Homepage type (Classic or Fluid) as well as allow them to select their Theme so that they can have the most optimal user experience.

### Define Personalization Option

Navigate to `Main Menu > PeopleTools > Personalization > Personalization Options`.  Open the _CSTM_ Option Category Level.  Add a new row to the grid and 
populate the fields under the _Definition_ tab as follows:

**User Option**: PSM_THEME  
**Description**: Portal Branding Theme  
**Option Category Group**: Custom Personalizations  
**Option Category**: Genral Options  
**User Option Type**: System  

[1]: /assets/images/2018/11/Option_Definition.png
[![Option Definition][1]][1]

Then populate the fields under the _Format_ tab as follows:

**Record (Table) Name**: PTBR_THEME  
**Field Name**: DESCR  

[2]: /assets/images/2018/11/Option_Format.png
[![Option Format][2]][2]

### Provision User Personalization Security

Navigate to `Main Menu > PeopleTools > Security > Permissions & Roles > Permission Lists`.  Open a Permission List of your choosing and select the 
Personalizations tab.  Add a new row to the Personalizartion Options grid and populate the fields as follows:

**Option Category Level**: Custom  
**Option Category Group**: Custom Personalizations  

[3]: /assets/images/2018/11/Permission_List.png
[![Permission List][3]][3]

Click the _Edit Options_ link to be taken to the Personalization Permissions page.  Check the _Allow User Option_ checkbox for the row with the _PSM_THEME_ User 
Option.

[4]: /assets/images/2018/11/Personalization_Permissions.png
[![Personalization Permissions][4]][4]

### Results

Now when a privileged user goes to the My Preferences page, they will have the ability to select a Portal Branding Theme from the list of configured Themes.

[5]: /assets/images/2018/11/My_Preferences2.png
[![My Preferences][5]][5]

* * *

The configuration done in this post will allow for developers to make use of the _GetUserOption_ function for the newly defined _PSM_THEME_ User Option. 
The PeopleCode would look like:

```java
Local String &sDescr = GetUserOption("CSTM", "PSM_THEME");
```

And the function will return the description of the Portal Branding Theme that the user has selected. 


In my next post, I will demonstrate how we can leverage the _GetUserOption_ function in a custom User Attribute so that we can conditionally apply the 
user’s selected Portal Branding Theme at runtime.