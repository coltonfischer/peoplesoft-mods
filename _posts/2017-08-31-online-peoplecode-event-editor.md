---
id: 1096
title: Online PeopleCode Event Editor
date: 2017-08-31T16:00:06+00:00
guid: https://www.peoplesoftmods.com/?p=1096
permalink: /tips-and-tricks/online-peoplecode-event-editor/
categories:
  - Tips and Tricks
---
I recently made a post discussing the [%metadata application package in PeopleSoft](https://www.peoplesoftmods.com/tips-and-tricks/understanding-the-metadata-application-package/).  I provided an overview of the package as well as examples of how to use it.  To support my research efforts in understanding this package, I made a simple web-based PeopleCode editor to edit PeopelCode events in the PIA.  My initial plan was to try build out a web IDE for accessing and modifying PeopleTools objects.  I have found myself busy working on other projects and wanted to at least share what I was able to create.  This project is simply an IScript that serves a basic interface to view and modify PeopleCode events using the %metadata application package.  In this post, I will walk through how to install and use this online PeopleCode editor project.  Please note that this project is a proof of concept and it is not intended to be used in any production PeopleSoft environments.

<!--more-->

<span style="text-decoration: underline;"><strong><a href="/assets/downloads/PSM_WEB_IDE.zip">CLICK HERE</a></strong></span> to download the Online PeopleCode Editor project.  Unzip the file and copy the project from file into App Designer.

The only setup that is needed to get this project working is the assignment of a Permission List. Login to the PIA and assign the PSM_WIDE Permission List to a role that you want to have access to the PeopleCode editor.

[<img class="alignnone size-full wp-image-1097" src="/assets/images/2017/08/Assign-Permission-List.png" alt="Assign Permission List" width="864" height="498" srcset="/assets/images/2017/08/Assign-Permission-List.png 864w, /assets/images/2017/08/Assign-Permission-List-300x173.png 300w, /assets/images/2017/08/Assign-Permission-List-768x443.png 768w, /assets/images/2017/08/Assign-Permission-List-659x380.png 659w" sizes="(max-width: 864px) 100vw, 864px" />](/assets/images/2017/08/Assign-Permission-List.png)

Using the user account with the appropriate role, navigate to Main Menu > PSM Projects > Web IDE.  Select the object type of PeopleCode program that you want to open.

[<img class="alignnone size-full wp-image-1098" src="/assets/images/2017/08/Select-Object-Type.png" alt="Select Object Type" width="492" height="320" srcset="/assets/images/2017/08/Select-Object-Type.png 492w, /assets/images/2017/08/Select-Object-Type-300x195.png 300w" sizes="(max-width: 492px) 100vw, 492px" />](/assets/images/2017/08/Select-Object-Type.png)

Populate the key values for the object type, select the name of the event, and then click the Open button.

[<img class="alignnone size-full wp-image-1099" src="/assets/images/2017/08/Select-Event-Name.png" alt="Select Event Name" width="537" height="440" srcset="/assets/images/2017/08/Select-Event-Name.png 537w, /assets/images/2017/08/Select-Event-Name-300x246.png 300w, /assets/images/2017/08/Select-Event-Name-464x380.png 464w" sizes="(max-width: 537px) 100vw, 537px" />](/assets/images/2017/08/Select-Event-Name.png)

If you provided key values to a valid PeopleCode program, then you should get presented with a garbled mess of an entire PeopleCode event represented as a raw string.  In this example, I have opened the PeopleCode on the Post Build event of User Profiles component.  At this point you can modify the code and click the Save button to save the changes.  If there are any syntax errors, then the IScript should respond with an error message.  However, the IScript is not coded to have robust error handling.

In this example, I added a simple message box to the end of the code and clicked the Save button.

[<img class="alignnone size-full wp-image-1100" src="/assets/images/2017/08/Edit-PeopleCode.png" alt="Edit PeopleCode" width="826" height="902" srcset="/assets/images/2017/08/Edit-PeopleCode.png 826w, /assets/images/2017/08/Edit-PeopleCode-275x300.png 275w, /assets/images/2017/08/Edit-PeopleCode-768x839.png 768w, /assets/images/2017/08/Edit-PeopleCode-348x380.png 348w" sizes="(max-width: 826px) 100vw, 826px" />](/assets/images/2017/08/Edit-PeopleCode.png)

Now you can navigate to the component to invoke the modified PeopleCode event to see the changes.

[<img class="alignnone size-full wp-image-1101" src="/assets/images/2017/08/Test-Changes.png" alt="Test Changes" width="870" height="666" srcset="/assets/images/2017/08/Test-Changes.png 870w, /assets/images/2017/08/Test-Changes-300x230.png 300w, /assets/images/2017/08/Test-Changes-768x588.png 768w, /assets/images/2017/08/Test-Changes-496x380.png 496w" sizes="(max-width: 870px) 100vw, 870px" />](/assets/images/2017/08/Test-Changes.png)

While this tool is really raw and cosmetically unappealing, I believe it does a decent job of demonstrating how the %metadata application package can be used to programmatically modify PeopleCode events.  I think this tool can be enhanced on many different fronts and would be interested in hearing if anyone is able to make this tool more usable.