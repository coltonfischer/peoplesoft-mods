---
id: 1350
title: Fluid Component Logging Techniques
date: 2019-10-04T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1350
permalink: /tips-and-tricks/fluid-component-logging-techniques/
tags:
  - JavaScript
  - JSON
  - Logging
categories:
  - Tips and Tricks
---

I find that effective usage of browser developer tools is crucial for productive PeopleSoft
development.  Having the ability to inspect and edit the CSS and HTML code directly within the
browser makes for a phenomenal developer experience.  A really handy feature of browser dev tools
is the JavaScript console.  The JavaScript console allows you to run lines of JavaScript against the
page currently loaded in the browser and reports the errors encountered as the browser tries to
execute your code.

### Simple Fluid Component Logging

For Fluid Component development, we can make use of the `AddOnLoadScript` function to execute console
log statements.  This can be useful in a Component debugging scenario where you want the ability to see
server-side log statement output in the browser.  Here is an example of this technique determining if the
user has the PeopleSoft Administrator Role on the Fluid Homepage Component PeopleCode:

```java
If IsUserInRole("PeopleSoft Administrator") Then
    AddOnLoadScript("console.log('User Has PS Admin Role');");
End-If;
```

And the output in the browser console:

[0]: /assets/images/2019/10/AddOnloadScript_Console_Log.png
[![AddOnLoadScript Console Log][0]][0]

### Advanced Fluid Component Logging

A nice feature of the browser console is that it provides support for viewing JSON objects by allowing
the developer to expand and collapse the objects and arrays within the JSON structure.

[1]: /assets/images/2019/10/JSON_Viewer.png
[![AddOnLoadScript Console Log][1]][1]

In a Fluid Component debugging context, the interactive JSON viewer can be used to view the Component
buffer data in a structured fashion.  The developer just needs to be able to convert PeopleSoft
Component data types (Rowsets, Rows, Records, Fields, etc.) to JSON objects to be logged in the console.

PeopleTools delivers the `JsonBuilder` Class that allows developers to build dynamic JSON objects.
The JSON objects can be converted to String and outputted in the browser console
using `AddOnLoadScript`.

Here is an example of converting a populated `PSOPRDEFN` Record to JSON and
logging it to the console:

```java
Local Record &rOprDefn = CreateRecord(Record.PSOPRDEFN);
&rOprDefn.OPRID.Value = %OperatorId;
&rOprDefn.SelectByKey();

Local JsonBuilder &jbData = CreateJsonBuilder();

&jbData.StartObject(&rOprDefn.Name);

Local integer &x;

For &x = 1 To &rOprDefn.FieldCount

    Local Field &fField = &rOprDefn.GetField(&x);
    &jbData.AddProperty(&fField.Name, &fField.Value);

End-For;

&jbData.EndObject(&rOprDefn.Name);

AddOnLoadScript("console.log(" | &jbData.ToString() | ");");
```

I am able to view and expand the JSON representation of the logged Record object in the browser console:

[2]: /assets/images/2019/10/Console_Log_PSOPRDEFN.png
[![AddOnLoadScript Console Log][2]][2]

In a real world Component debugging scenario, it is not practical to write the several additional
lines of code for the sake of getting debugging information written to the browser console.  Instead,
utility Application Classes can be written to transform PeopleSoft data types to JSON objects to
abstract unnecessary logic (loops, etc.) when the need for object conversion arises.  The logging
implementation logic (`AddOnLoadScript`, etc.) can also be abstracted in an Application Class to
allow for simple invocation of the logging statements.