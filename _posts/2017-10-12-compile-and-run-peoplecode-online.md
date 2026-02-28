---
id: 1182
title: Compile and Run PeopleCode Online
date: 2017-10-12T19:00:06+00:00
guid: https://www.peoplesoftmods.com/?p=1182
permalink: /utilities/compile-and-run-peoplecode-online/
tags:
 - Metadata API
 - PeopleCode
categories:
 - Utilities
---

There are many times where I come across some sample PeopleCode on the internet and I want to execute the PeopleCode in my environment to see the output. The route I take to test drive some PeopleCode usually involves me opening up an existing object in App Designer, pasting in the sample code, and then going to the PIA to see the results. I find this process rather tedious to perform just to see the output of some sample code. Not to mention, I have to make sure I go back into App Designer and clean up the object I modified. More often than not, I already have a PeopleSoft session open in my web browser when I am exploring PeopleCode online. So what I decided to do was build an online utility for compiling and running PeopleCode directly in the PIA. In this post, I will share this helpful utility for anyone that is interested in this functionality.

This solution follows a recurring theme of using the _%metadata_ Application Package to update PeopleTools-managed objects through PeopleCode. I have a couple of recent posts on this topic, one is on [understanding the %metadata Application Package](/tips-and-tricks/understanding-the-metadata-application-package/) and the other is about using the _%metadata_ Application Package to create an [online PeopleCode event editor](/tips-and-tricks/online-peoplecode-editor-project/). The technicalities of this solution are very similar to that of the online PeopleCode editor project.

[Download the App Designer](/Development/PSM_RUN_PC.zip) to  project for this utility. Unzip the file and copy the project from file into App Designer.

The only setup that is needed to use this utility is the assignment of a Permission List. Login to the PIA and assign the _PSM_RUN_PC_ Permission List to a role that you want to have access to the PeopleCode executer. The user that gets access to the PeopleCode executer must also have the necessary privileges to update Application Package objects. This is because the utility updates an Application Class method with the inputted PeopleCode to run.

After performing the security setup, you can login as the privileged user. The utility can be found by navigating to `Main Menu > PSM Projects > Run PeopleCode`.

[1]: /assets/images/2017/10/Run-PeopleCode.png
[![Run PeopleCode][1]][1]

On this page you will be greeted with an empty input box, a save button, and a run button. The usage of the utility is simple: Paste in some PeopleCode, click the save button to compile the PeopleCode, and then click the run button to execute the PeopleCode.

In this example I wanted to run some sample PeopleCode to output the _PS_HOME_ and _PS_CFG_HOME_ environment variables.

```java
Local string &sOutput;
&sOutput = "PS_HOME: " | GetEnv("PS_HOME") | Char(10);
&sOutput = &sOutput | "PS_CFG_HOME: "" | GetEnv("PS_CFG_HOME");
MessageBox(0, "", 0, 0, &sOutput);
```

After pasting in the code and clicking the save button, the PeopleCode is ready to be executed. In this example clicking the run button results in a messagebox displaying the environment variables.

[2]: /assets/images/2017/10/Output.png
[![Output][2]][2]

### How it Works

The PeopleCode behind the save button is responsible for updating the Application Class PeopleCode of the generic _RunPC_ Application Class within the _PSM_RUN_PC_ Application Package. The code uses the _%metadata_ Application Package to compile and save the Application Class PeopleCode. Here is the code behind the save button that updates the Application Class PeopleCode using _%metadata_:

<pre><span style="color: #0000ff;">import</span> %metadata:Key;
<span style="color: #0000ff;">import</span> %metadata:PeopleCodeProgram:*;
<span style="color: #0000ff;">import</span> %metadata:AppPackageDefn:*;

<span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager &mgrPeopleCodeProgram = <span style="color: #0000ff;">create</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager();
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">number</span> &int1, &int2;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &strErr;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bSaved, &bResult;
<span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram &defn;

<span style="color: #339966;">/* Create the key to reference the generic RunPC Application Class */</span>
<span style="color: #0000ff;">Local</span> %metadata:Key &key = <span style="color: #0000ff;">create</span> %metadata:Key();
&key.AddItem(key:Class_ApplicationPackage, <span style="color: #ff0000;">"PSM_RUN_PC"</span>);
&key.AddItem(key:Class_ApplicationClass, <span style="color: #ff0000;">"RunPC"</span>);
&key.AddItem(key:Class_Method, <span style="color: #ff0000;">"OnExecute"</span>);

<span style="color: #339966;">/* Display an error if the object definition doesn't exist */</span>
<span style="color: #0000ff;">If</span> <span style="color: #0000ff;">Not</span> &mgrPeopleCodeProgram.DefnExists(&key) <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, <span style="color: #ff0000;">"Definition Error"</span>);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #339966;">/* Get an updateable object definition so that we can overwrite the existing PeopleCode */</span>
&defn = &mgrPeopleCodeProgram.GetDefnToUpdate(&key);

<span style="color: #339966;">/* Insert the new PeopleCode into the Run method within the boilerplate Application Class code */</span>
&appClassText = <span style="color: #ff0000;">"class RunPC method Run();end-class; method Run "</span> | PSM_RUN_PC_WK.PCTEXT.<span style="color: #0000ff;">Value</span> | <span style="color: #ff0000;">" end-method;"</span>;

<span style="color: #339966;">/* Attempt to compile the Application Class PeopleCode */</span>
&bResult = &defn.UpdateProgram(&appClassText, &strErr, &int1, &int2);

<span style="color: #339966;">/* Display an error if the code failed to compile */</span>
<span style="color: #0000ff;">If</span> <span style="color: #0000ff;">Not</span> (&bResult) <span style="color: #0000ff;">Then</span>
 <span style="color: #339966;">/* I think &int1 and &int2 can be used to determine where exactly in the code the error occured */</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, <span style="color: #0000ff;">MsgGetText</span>(158, 20153, <span style="color: #ff0000;">"PeopleCode Error"</span>) | <span style="color: #ff0000;">" "</span> | &strErr);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #339966;">/* Attempt to update the object definition */</span>
&bSaved = &defn.UpdateDefn();

<span style="color: #339966;">/* Display an error if the object definition failed to update */</span>
<span style="color: #0000ff;">If</span> <span style="color: #0000ff;">Not</span> (&bSaved) <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(158, 20152, <span style="color: #ff0000;">"Error saving new PeopleCode."</span>);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #339966;">/* Enable the Run button and disable the Save button */</span>
PSM_RUN_PC_WK.BUTTON1.Enabled = <span style="color: #0000ff;">True</span>;
PSM_RUN_PC_WK.BUTTON.Enabled = <span style="color: #0000ff;">False</span>;</pre>

If the provided PeopleCode is invalid, then an error would be displayed to the user when they click the save button. If the code compiles and saves successfully, then the following code behind the run button is used to execute the updated Application Class method that contains the PeopleCode to run:

<pre><span style="color: #0000ff;">import</span> PSM_RUN_PC:RunPC;

<span style="color: #0000ff;">Local</span> PSM_RUN_PC:RunPC &oRunPC = <span style="color: #0000ff;">create</span> PSM_RUN_PC:RunPC();

<span style="color: #339966;">/* Execute the updated Run method */</span>
<span style="color: #0000ff;">try</span>
 &oRunPC.Run();
<span style="color: #0000ff;">catch</span> Exception &e
 <span style="color: #339966;">/* Enable the Save button and disable the Run button */</span>
 PSM_RUN_PC_WK.BUTTON.Enabled = <span style="color: #0000ff;">True</span>;
 PSM_RUN_PC_WK.BUTTON1.Enabled = <span style="color: #0000ff;">False</span>;
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, &e.ToString());
<span style="color: #0000ff;">end-try</span>;

<span style="color: #339966;">/* Enable the Save button and disable the Run button */</span>
PSM_RUN_PC_WK.BUTTON.Enabled = <span style="color: #0000ff;">True</span>;
PSM_RUN_PC_WK.BUTTON1.Enabled = <span style="color: #0000ff;">False</span>;</pre>

* * *

While this is a helpful utility, it should be well understood that enabling this type of functionality for users in the PIA could result is some serious security implications. I would also like to point out that the delivered Application Class Tester utility within Enterprise Components could potentially be leveraged to perform similar functionality as the utility presented in this post. If you are interested in playing with that utility, then it can be found under `Main Menu > Enterprise Components > Component Configurations > Application Class Tester`.