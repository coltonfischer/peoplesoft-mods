---
id: 1342
title: User Preference-Based Homepage Types
date: 2018-11-14
guid: http://www.peoplesoftmods.com/?p=1342
permalink: /ux/user-preference-based-homepage-types/
tags:
  - User Personalizations
categories:
  - User Experience
---

PeopleSoft allows for administrative users to configure and create Personalization Options that allow end users to personalize their application experience via 
the My Preferences page.  Some example user preferences are menu sort order, spell check, and autocomplete.  While there are many delivered 
Personalization Options, not all of them are allowed to be edited by end users.  Examples of uneditable preferences include the PC and Tablet Homepage 
types (Fluid or Classic).  If you want to allow a certain (Permission List-based) audience the ability to specify their Homepage types, then there are only a 
couple of steps that need to be performed to enable this functionality.

### Configure User Personalizations

Navigate to `Main Menu > PeopleTools > Personalization > Personalization Options`.  Open the _PPTL_ Option Category Level.  Scroll down to the _HPPC_ and _HPTABLET_ 
rows and change the User Option Type from _Functional_ to _System_.

[1]: /assets/images/2018/11/Personalization_Options.png
[![Personalization Options][1]][1]

### Provision User Personalization Security

Generally, User Personalization security can be configured through the Permission Lists page in the PIA.  However, there is Page Activate PeopleCode on the 
_PLIST_OPTN_SEC_ page that hides the _HPPC_ and _HPTABLET_ User Personalizations.  This prevents the provisioning of the User Personalization security in the PIA, but 
there is nothing preventing the Permission List assignment via direct database SQL.  The following SQL will successfully provision the User Personalization 
security:

```sql
INSERT INTO PSAUTHOPTN VALUES ('PTPT1000', 'PPTL', 'HPPC');
INSERT INTO PSAUTHOPTN VALUES ('PTPT1000', 'PPTL', 'HPTABLET');
 ```

Note: You can change the Permission List name from PTPT1000 to any Permission List.

### Results

Now when a user with the _PTPT1000_ Permission List goes to the My Preferences page, they will have the ability to specify _Classic_ or _Fluid_ for the PC 
Homepage and Tablet Homepage Preferences.

[2]: /assets/images/2018/11/My_Preferences.png
[![My Preferences][2]][2]

* * *

Having the ability to specify Homepage types via My Preferences is nice, but in practice, this alone is not enough to create an optimally preferred user 
experience.  If a user opts to have a Classic Homepage type, then they are most likely going to prefer to have the entire experience (Theme) be Classic. 
The same is true for Fluid Homepage/Theme.  This is where User Attribute Theme assignments can help.  However, the delivered Role and Permission List-based 
Theme assignment will not be sufficient to solve this particular requirement.  

I will solve this requirement in my next post by demonstrating how to create a custom Personalization Option to allow end users to select a Portal Branding Theme 
that they want to use. This custom Personalization Option will then be used is a Custom User Attribute to allow for Preference-based Theme assignments.