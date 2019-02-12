---
id: 884
title: Making Objects Accessible in the Branding Framework
date: 2017-03-31T18:00:42+00:00
guid: https://www.peoplesoftmods.com/?p=884
permalink: /tips-and-tricks/making-objects-accessible-in-the-branding-framework/
categories:
  - Tips and Tricks
---
The Branding Framework pages are really useful because they provide the ability to add and update branding-related objects from within the PIA. These pages provide a convenient alternative to having to open up App Designer to make simple changes to HTML, JavaScript, Images, or Style Sheets. One limitation that I have found with the Branding Framework is that you cannot use it to modify objects that were not created with the framework. For example, if I want to make a change to the PT_COMMON HTML definition, then I would I have to make this change in App Designer. It turns out that there is a delivered PeopleCode library that allows us to manage the Branding Framework objects. I would like to demonstrate how this library can be used to add an existing object to the Branding Framework so that the object can be modified from the PIA.

<!--more-->

I will start off by creating a new HTML definition in App Designer.

[<img class="alignnone size-full wp-image-885" src="/assets/images/2017/03/New_Object.png" alt="New_Object" width="621" height="374" srcset="/assets/images/2017/03/New_Object.png 621w, /assets/images/2017/03/New_Object-300x181.png 300w" sizes="(max-width: 621px) 100vw, 621px" />](/assets/images/2017/03/New_Object.png)

As I mentioned above, I cannot simply go to the Branding Objects page (PeopleTools > Portal > Branding > Branding Objects) in the PIA to modify the contents of this object. When I go to the Branding Objects page at this point, I am presented with no modifiable objects.

[<img class="alignnone size-full wp-image-886" src="/assets/images/2017/03/Branding_Objects.png" alt="Branding_Objects" width="381" height="280" srcset="/assets/images/2017/03/Branding_Objects.png 381w, /assets/images/2017/03/Branding_Objects-300x220.png 300w" sizes="(max-width: 381px) 100vw, 381px" />](/assets/images/2017/03/Branding_Objects.png)

I made a simple page to get the newly created object imported into the Branding Framework. The page has an input field and a button.

[<img class="alignnone size-full wp-image-887" src="/assets/images/2017/03/Test_Page.png" alt="Test_Page" width="703" height="500" srcset="/assets/images/2017/03/Test_Page.png 703w, /assets/images/2017/03/Test_Page-300x213.png 300w, /assets/images/2017/03/Test_Page-534x380.png 534w" sizes="(max-width: 703px) 100vw, 703px" />](/assets/images/2017/03/Test_Page.png)

This page will take an HTML object name in the input field and upload the object to the Branding Framework on the button click.

[<img class="alignnone size-full wp-image-888" src="/assets/images/2017/03/Input_Value.png" alt="Input_Value" width="598" height="226" srcset="/assets/images/2017/03/Input_Value.png 598w, /assets/images/2017/03/Input_Value-300x113.png 300w" sizes="(max-width: 598px) 100vw, 598px" />](/assets/images/2017/03/Input_Value.png)

When the user inputs the HTML object name into the input field and clicks the OK button, a small piece of PeopleCode fires.

[<img class="alignnone size-full wp-image-889" src="/assets/images/2017/03/Save_Object.png" alt="Save_Object" width="873" height="330" srcset="/assets/images/2017/03/Save_Object.png 873w, /assets/images/2017/03/Save_Object-300x113.png 300w, /assets/images/2017/03/Save_Object-768x290.png 768w" sizes="(max-width: 873px) 100vw, 873px" />](/assets/images/2017/03/Save_Object.png)

<pre>import PTBR_BRANDING:BrandingObj;

/* Populate the user inputted object name */
Local string &sHTMLObjName = PSM_WRK0.CONTNAME.Value;

/* Reference the BrandingObj class */
Local PTBR_BRANDING:BrandingObj &objBranding;
&objBranding = create PTBR_BRANDING:BrandingObj();

/* Call the new method with the object name, object type (H-HTML,I-Image,J-JavaScript,S-Style), user ID, and date */
&objBranding.new(&sHTMLObjName, "J", %UserId, %Datetime);

/* Call the save method to save the object to the Branding Framework */
If (&objBranding.save()) Then
 MessageBox(0, "", 0, 0, "Saved");
Else
 MessageBox(0, "", 0, 0, "Save Failed");
End-If;</pre>

This PeopleCode uses the BrandingObj class of the PTBR_BRANDING application package to import the HTML object definition as a JavaScript object in the Branding Framework. The BrandingObj class can be used in the same fashion for uploading other object types such as images or style sheets into the Branding Framework.

Now when I go back to the Branding Objects page in the PIA, I can see my newly imported JavaScript object.

[<img class="alignnone size-full wp-image-890" src="/assets/images/2017/03/New_Branding_Object.png" alt="New_Branding_Object" width="752" height="327" srcset="/assets/images/2017/03/New_Branding_Object.png 752w, /assets/images/2017/03/New_Branding_Object-300x130.png 300w" sizes="(max-width: 752px) 100vw, 752px" />](/assets/images/2017/03/New_Branding_Object.png)

I am no longer bound to only being able to update this object’s content in App Designer.  I can now modify this object from within the PIA.