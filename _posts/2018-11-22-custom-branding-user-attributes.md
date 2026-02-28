---
id: 1344
title: Custom Branding User Attributes
date: 2018-11-22
guid: https://www.peoplesoftmods.com/?p=1344
permalink: /ux/custom-branding-user-attributes/
tags:
  - User Attributes
  - Branding Framework
categories:
  - User Experience
---

User Attributes are a nice feature of the PeopleTools Branding framework. User Attributes allow us to conditionally apply Portal Branding Themes in a 
context-sensitive manner. Examples of delivered User Attributes types are the Role and Permission List types which allows 
for [Role and Permission List based theme assignments](https://pe0ples0ft.blogspot.com/2016/08/peopletools-branding-role-based-themes.html). The PeopleTools 
Branding framework provides the ability to define your own custom User Attribute Types. I will outline the required steps to define a custom User Attribute 
that allows for user preference-based theme assignments. This post is a continuation of my previous two posts that describe how Personalization Options can 
be leverage to provide your users with a preference-based experience.  

The end goal functionality is to allow end-users to specify a theme (Tangerine, Fluid, Red, Blue, etc.) that they prefer and for the system to 
conditionally assign the theme based on the user’s preferred theme. In [my previous post](/ux/creating-personalization-options/) I covered how to enable 
the _Portal Branding Theme_ selection drop down on the end-user _My Preferences_ page:

[5]: /assets/images/2018/11/My_Preferences2.png
[![My Preferences][5]][5]

All that is needed now is some Application Class PeopleCode and a little bit of configuration to connect the user preference selection to a custom User Attribute.

### The Code

The first step in creating a custom User Attribute is to create a class that extends the delivered _PTBR_BRANDING:UserAttributes:BaseUserAttribute_ base class. 
The custom class will implement the _getPromptViewName_ and _validateValue_ methods of the base class.  The _getPromptViewName_ method returns the table 
name (_PTBR_THEME_) to be used in the prompt on the Assign Themes page and the _validateValue_ method is responsible for performing the logic in determining 
if a particular theme needs to be assigned to a user during a session.  As you can see in the code below, the _GetUserOption_ function is used to read in the 
user’s preferred theme description that ultimately determines the theme assignment.

```java
import PTBR_BRANDING:UserAttributes:BaseUserAttribute;

class PreferenceBasedUserAttribute extends PTBR_BRANDING:UserAttributes:BaseUserAttribute
   method PreferenceBasedUserAttribute(&pId As string);
   
   method getPromptViewName() Returns string;
   method validateValue(&pValue As string) Returns boolean;
   
end-class;


method PreferenceBasedUserAttribute
   /+ &pId as String +/
   
   %Super = create PTBR_BRANDING:UserAttributes:BaseUserAttribute(&pId);
   
   %This.setUserAttributeType("PreferenceBasedUserAttribute");
   
end-method;


method getPromptViewName
   /+ Returns String +/
   /+ Extends/implements PTBR_BRANDING:UserAttributes:BaseUserAttribute.getPromptViewName +/
   
   Return Record.PTBR_THEME;
   
end-method;


method validateValue
   /+ &pValue as String +/
   /+ Returns Boolean +/
   /+ Extends/implements PTBR_BRANDING:UserAttributes:BaseUserAttribute.validateValue +/
   
   Local Record &rBrandingTheme = CreateRecord(Record.PTBR_THEME);
   &rBrandingTheme.PTBR_THEME_ID.Value = &pValue;
   &rBrandingTheme.SelectByKey();
   
   Return (GetUserOption("CSTM", "PSM_THEME") = &rBrandingTheme.DESCR.Value);
   
end-method;
```

### The Configuration

Once the custom User Attribute PeopleCode has been placed in an Application Class, the User Attribute class will need to be registered in PIA.  Navigate 
to `Main Menu > PeopleTools > Portal > Branding > System Data > Define User Attribute Types`.  Define the _PREFERENCE_BASED_ User Attribute type and populate 
the Supporting Application Class fields accordingly.  

[1]: /assets/images/2018/11/Define_User_Attribute.png
[![Define User Attribute][1]][1]

Once this is complete, you can navigate to `Main Menu > PeopleTools > Portal > Branding > Assign Themes`.  You should have a new _Preference_ value in the 
Attribute Type drop down on this page.  Add rows to the User Attribute Based Theme Assignments grid for each of the existing themes and ensure to set 
the _Attribute Value_ and _Theme Value_ to the same value for each row.  

[2]: /assets/images/2018/11/Assign_Branding_Theme.png
[![Assign Branding Theme][2]][2]

The system will loop over the rows in this grid at sign in time and it will call the _validateValue_ method of the custom User Attribute Application Class to 
determine if the particular theme needs to be assigned to the user’s session.

* * *

When a user selects a theme in My Preferences, the selected theme should get applied the next time the user logs into the system.