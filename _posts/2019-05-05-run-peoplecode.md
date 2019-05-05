---
id: 1347
title: Run PeopleCode
date: 2019-05-05T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1347
permalink: /utilities/run-peoplecode/
tags:
  - PeopleCode
categories:
  - Utilities
---

Running arbitrary PeopleCode statements and scripts in the PIA has always been a desirable feature for me.  I previously scratched 
this itch by writing a proof of concept utility that allows you 
to [Compile and Run PeopleCode Online](/utilities/compile-and-run-peoplecode-online/).  While the utility is 
helpful, it is completely decoupled from the Online PeopleCode Editor that I play in.  I decided to incorporate the Run PeopleCode 
functionality into the Online PeopleCode Editor project.  You can check out the [GitHub repository](https://github.com/coltonfischer/ps-web-ide) 
to download the latest version of the Online PeopleCode Editor project that contains this functionality.

To use the Run PeopleCode functionality in the Online PeopleCode Editor you must create an App Class that will contain the code to be run.  Unfortunately 
the current version of the Online PeopleCode Editor does not allow for new object creation, so you will need to use App Designer to 
create the new App Class.  

The Class will need to implement `PSM_RUN_PC:IRunner` Interface and have a Run method that returns a String.  Here is an example:

```java
import PSM_RUN_PC:IRunner;

class Test implements PSM_RUN_PC:IRunner
   method Run() Returns string;
end-class;

method Run
   /+ Returns String +/
   /+ Extends/implements PSM_RUN_PC:IRunner.Run +/
   
   /* Write PeopleCode to be ran here */
   Local string &sOutput;
   &sOutput = "Tools Version: " | %ToolsRelease;
   
   Return &sOutput;
   
end-method;
```

After the App Class is created you can use the Online PeopleCode Editor to open and edit the App Class PeopleCode.  The editor will 
present a “Run” button if it detects an App Class that implements `PSM_RUN_PC:IRunner` Interface.

[1]: /assets/images/2019/05/Run_Button.png
[![Run PeopleCode][1]][1]

Clicking the Run button will invoke the Run method of the App Class and it will display the returned String in a modal window.  Here 
is the output of the Run method of the App Class PeopleCode above:

[2]: /assets/images/2019/05/Output.png
[![PeopleCode Program Output][2]][2]

This example simply runs a PeopleCode statement to get the PeopleTools version for the environment; however the PeopleCode programs 
contained in the Run method can be as complex as the developer desires them to be.