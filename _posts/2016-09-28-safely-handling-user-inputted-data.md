---
id: 673
title: Safely Handling User Inputted Data
date: 2016-09-28T15:46:30+00:00
guid: https://www.peoplesoftmods.com/?p=673
permalink: /tips-and-tricks/safely-handling-user-inputted-data/
categories:
  - Tips and Tricks
---
It should be well understood that you should never trust user input in your application.  As an application developer, I always try my hardest to enforce very strict rules when accepting and outputting user inputted data. You can never really be too careful when it comes to handling data that you do not know (or can’t trust) the source of. Fortunately, PeopleCode is very robust in terms of providing built-in functions to safely handle the input and output of data.  I would like to demonstrate an example of how a malicious user can execute a stored cross-site scripting attack on an insecure custom application within my PeopleSoft system. I will then show how to mitigate this attack by hardening the security of my custom application with a built-in PeopleCode function.

<!--more-->

**The Custom Application:**

The custom application is very simple.  All it does is say hello to the user.  The code gets the user’s preferred first name from the database and outputs “Hello <name>!”. The text is outputted in an HTML area on the page so that that the text can be made bold and blue.  Here is the page activate PeopleCode:

[<img class="alignnone size-full wp-image-674" src="/assets/images/2016/09/1_Code_Before.png" alt="1_Code_Before" width="808" height="176" srcset="/assets/images/2016/09/1_Code_Before.png 808w, /assets/images/2016/09/1_Code_Before-300x65.png 300w, /assets/images/2016/09/1_Code_Before-768x167.png 768w" sizes="(max-width: 808px) 100vw, 808px" />](/assets/images/2016/09/1_Code_Before.png)

And here is an example of the output for a user named John:

[<img class="alignnone size-full wp-image-675" src="/assets/images/2016/09/2_Output.png" alt="2_Output" width="829" height="313" srcset="/assets/images/2016/09/2_Output.png 829w, /assets/images/2016/09/2_Output-300x113.png 300w, /assets/images/2016/09/2_Output-768x290.png 768w" sizes="(max-width: 829px) 100vw, 829px" />](/assets/images/2016/09/2_Output.png)

**The Malicious User:**

Let’s suppose John is a malicious user and he notices that the application is displaying his name to him.  He wants to attempt a stored cross-site scripting attack on the PeopleSoft application and he wants to use his first name as the driver of his payload. John will simply use self-service to change his preferred name.

[<img class="alignnone size-full wp-image-676" src="/assets/images/2016/09/3_Pref_Name_Page.png" alt="3_Pref_Name_Page" width="850" height="513" srcset="/assets/images/2016/09/3_Pref_Name_Page.png 850w, /assets/images/2016/09/3_Pref_Name_Page-300x181.png 300w, /assets/images/2016/09/3_Pref_Name_Page-768x464.png 768w, /assets/images/2016/09/3_Pref_Name_Page-630x380.png 630w" sizes="(max-width: 850px) 100vw, 850px" />](/assets/images/2016/09/3_Pref_Name_Page.png)

He will input a script as his preferred first name.

[<img class="alignnone size-full wp-image-677" src="/assets/images/2016/09/4_Malicious_Input.png" alt="4_Malicious_Input" width="837" height="511" srcset="/assets/images/2016/09/4_Malicious_Input.png 837w, /assets/images/2016/09/4_Malicious_Input-300x183.png 300w, /assets/images/2016/09/4_Malicious_Input-768x469.png 768w, /assets/images/2016/09/4_Malicious_Input-622x380.png 622w" sizes="(max-width: 837px) 100vw, 837px" />](/assets/images/2016/09/4_Malicious_Input.png)

And then the application will save his input.

[<img class="alignnone size-full wp-image-678" src="/assets/images/2016/09/5_Injected_Script.png" alt="5_Injected_Script" width="858" height="522" srcset="/assets/images/2016/09/5_Injected_Script.png 858w, /assets/images/2016/09/5_Injected_Script-300x183.png 300w, /assets/images/2016/09/5_Injected_Script-768x467.png 768w, /assets/images/2016/09/5_Injected_Script-625x380.png 625w" sizes="(max-width: 858px) 100vw, 858px" />](/assets/images/2016/09/5_Injected_Script.png)

**The Bad Data:**

In this example, a non-malicious script was injected into the application.  The script will just output an alert box with the text “123”. It should be clear that this script could’ve just as easily been crafted in a way to cause harm.  Regardless, the script is now stored in the database at this point.

[<img class="alignnone size-full wp-image-679" src="/assets/images/2016/09/6_Bad_Data_Stored.png" alt="6_Bad_Data_Stored" width="864" height="239" srcset="/assets/images/2016/09/6_Bad_Data_Stored.png 864w, /assets/images/2016/09/6_Bad_Data_Stored-300x83.png 300w, /assets/images/2016/09/6_Bad_Data_Stored-768x212.png 768w" sizes="(max-width: 864px) 100vw, 864px" />](/assets/images/2016/09/6_Bad_Data_Stored.png)

This is half of the battle when it comes to performing a stored cross-site scripting attack. Since the script is now in the database, there is the possibility of the script being executed in the application if proper measures are not taken when outputting the data.  It is also worth noting that if this data gets fed to third-party systems, then the third-party systems will be at risk as well.

**Executing the Script:**

At this point, John will navigate back to the custom application that outputs his name to see if his stored script will execute.  To his delight, he found that the script executes as soon as the page loads.

[<img class="alignnone size-full wp-image-680" src="/assets/images/2016/09/7_XSS.png" alt="7_XSS" width="1072" height="380" srcset="/assets/images/2016/09/7_XSS.png 1072w, /assets/images/2016/09/7_XSS-300x106.png 300w, /assets/images/2016/09/7_XSS-768x272.png 768w, /assets/images/2016/09/7_XSS-1024x363.png 1024w" sizes="(max-width: 1072px) 100vw, 1072px" />](/assets/images/2016/09/7_XSS.png)

It is obvious that this example is not very threatening because the script will only run for John and it is just outputting “123”.  Anytime someone else navigates to this page, they will see their own name and not John’s name. Therefore, the script shouldn’t get executed on other users.  To make this more interesting, John could’ve set PeopleSoft meta-HTML in his name to try to extract information from the system, but that will not be demonstrated here.

**Securing the Custom Application:**

Now I will demonstrate how this type of attack can be mitigated with the use of the delivered EscapeHTML function in PeopleCode.  When outputting the name extracted from the database, the EscapeHTML function can be used to prevent the string from being interpreted as actual HTML in the HTML area that is displaying it.

[<img class="alignnone size-full wp-image-681" src="/assets/images/2016/09/8_Code_After.png" alt="8_Code_After" width="823" height="178" srcset="/assets/images/2016/09/8_Code_After.png 823w, /assets/images/2016/09/8_Code_After-300x65.png 300w, /assets/images/2016/09/8_Code_After-768x166.png 768w, /assets/images/2016/09/8_Code_After-820x178.png 820w" sizes="(max-width: 823px) 100vw, 823px" />](/assets/images/2016/09/8_Code_After.png)

This will result in the following output when a script (or any other HTML) is stored in the name field.

[<img class="alignnone size-full wp-image-682" src="/assets/images/2016/09/9_Output.png" alt="9_Output" width="834" height="306" srcset="/assets/images/2016/09/9_Output.png 834w, /assets/images/2016/09/9_Output-300x110.png 300w, /assets/images/2016/09/9_Output-768x282.png 768w" sizes="(max-width: 834px) 100vw, 834px" />](/assets/images/2016/09/9_Output.png)

**Conclusion:**

I realize that this was a brutally oversimplified example to demonstrate the idea of not trusting user inputted data.  However, it has been my experience that safely handling user inputted data is not exactly a common practice for some application developers.  Much like the EscapeHTML function, PeopleCode offers many great delivered functions to help protect our applications. No matter how overkill it may seem, we should always use these provided functions when developing custom applications. Another takeaway is that you should never assume that data has been pre-sanitized. The original custom application code wrongfully assumed that the preferred names were being stored in a safe state. This was unfortunately not the case, and this assumption is what ultimately opened up the application to the attack.