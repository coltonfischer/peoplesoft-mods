---
id: 1280
title: PeopleCode Syntax Highlighting in Ace Editor
date: 2018-02-15T18:48:18+00:00
guid: http://www.peoplesoftmods.com/?p=1280
permalink: /tips-and-tricks/peoplecode-syntax-highlighting-in-ace-editor/
peoplecode:
  - |
    /*-------------------------------------------------------------------*/ /* The Verify_Totp function can be used in SignOn PeopleCode to      */ /* validate inputted Time-Based One-Time Passwords (TOTPs) provided  */ /* at login. Only users with the PSM_TOTP_USER Permission List will  */ /* be required to provide a TOTP. This function will not apply to    */ /* other users. If a TOTP user provides an invalid TOTP, then they   */ /* will not be authenticated into the system even if they provide    */ /* correct username anmd password credentials.                       */ /*-------------------------------------------------------------------*/  Function Verify_Totp()        /* If the user does not have the TOTP User PL, then let them proceed */    If Not (IsUserInPermissionList("PSM_TOTP_USER")) Then       Return;    End-If;        /* If the posted totp parameter is blank, then set auth result to false */    Local string &sTotpInput = %Request.GetParameter("totp");        If None(&sTotpInput) Then       SetAuthenticationResult( False);    End-If;        /* TODO: Get the user's secret key from the DB */    Local string &sSecretKey = "CIFNEMSE4OM73YPS";        /* Validate the totp value */    Local JavaObject &manager = CreateJavaObject("javax.script.ScriptEngineManager");    Local JavaObject &engine = &manager.getEngineByName("JavaScript");    Local string &program = GetHTMLText(HTML.PSM_SHA) | GetHTMLText(HTML.PSM_TOTP);        &engine.put("userKey", &sSecretKey);    &engine.put("userOtp", &sTotpInput);    &engine.put("w", 5); /* window: 5 * 30s = 2.5 min */    &engine.eval(&program);        /* If the totp value is invalid, then set auth result to false */    Local string &result = &engine.get("result").toString();    If &result <> "true" Then       SetAuthenticationResult( False);    End-If;     End-Function;
categories:
  - Tips and Tricks
  - Utilities
---
Ace is an embeddable code editor written in JavaScript.  My first exposure to the <a href="https://ace.c9.io/" target="_blank">Ace Editor</a> was when I started to use the <a href="https://c9.io" target="_blank">Cloud9 IDE</a> for non-PeopleSoft development.  I like using web-based tools because they prevent me from being tied to a particular machine to do work.  With tools like Cloud9, I can develop software from any one of my internet-connected devices.  PeopleSoft development is a bit different than developing software in other languages as App Designer is needed to edit PeopleCode programs. I would rather not have to always rely on a client-based application to edit PeopleCode.  This is the reason that I embarked on writing a JavaScript-based PeopleCode editor powered by Ace.  The Ace Editor provides many desirable features that can be found in most modern editors and it also allows for language-specific syntax highlighting. Today I would like to share the PeopleCode syntax mode that I created for the Ace Editor.

<!--more-->

Usage of the PeopleCode syntax highlighter in the Ace Editor is really simple.  Below is an example that demonstrates the functionality.
  
<!---->


  


The syntax highlighter is not perfect, but it is definitely a step up from no highlighting at all.  I hope to provide updates to the highlighting rules as I have time to <a href="https://github.com/coltonfischer/PeopleCode-Ace-Editor" target="_blank">the GitHub repository here</a>.  I certainly welcome any community contributed enhancements to this repository as well.

While PeopleCode syntax highlighting with the Ace Editor is helpful for displaying read-only PeopleCode snippets online, this functionality really becomes useful when used in an online PeopleCode editor that is capable of making real time code updates to the application.  I did a [post last year](http://www.peoplesoftmods.com/tips-and-tricks/online-peoplecode-editor-project/) discussion the idea of exposing the %metadata Application Package as an API of sorts to support the backend of an online PeopleCode editor.  While this was a solid proof of concept, the demonstrated editor in that post was undoubtedly hideous.  Since then I have added a slew of changes with the most important one being the incorporation of the Ace Editor.  Here is a screen shot of the progress that has been made to the online PeopleCode editor project:

[<img class="size-full wp-image-1297" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor.png" alt="Online PeopleCode Editor" width="1163" height="882" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor.png 1163w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor-300x228.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor-768x582.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor-1024x777.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor-501x380.png 501w" sizes="(max-width: 1163px) 100vw, 1163px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/02/Online-PeopleCode-Editor.png)

As you can probably tell, the Ace Editor with PeopleCode syntax highlighting makes the online PeopleCode editor much more usable.   I am excited to share and document the new features and functionality of the online PeopleCode editor project in a future post.