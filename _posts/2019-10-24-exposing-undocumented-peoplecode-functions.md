---
id: 1351
title: Exposing Undocumented PeopleCode Functions
date: 2019-10-24T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1351
permalink: /tips-and-tricks/exposing-undocumented-peoplecode-functions/
tags:
  - JavaScript
  - Java
  - JSON
categories:
  - Tips and Tricks
---

There are some really useful built-in PeopleCode functions that are available to be consumed, but are not documented
in PeopleBooks.  Good examples of these functions include the `CreateJsonXxx` functions used for creating and parsing
JSON structures.  Developers can expose delivered usages of undocumented functions by doing “Find In” searches in App
Designer.  This method of uncovering undocumented functions is rather tedious and only exposes the undocumented functions
that are being used in delivered code.  It is quite possible that there are delivered undocumented functions that are not
being used in any delivered code.  How can we expose these undocumented, unused functions?

The answer is to use Java to perform type introspection on the delivered `PeopleSoft.PeopleCode.Func` Java Class.  The Func
Java Class is mentioned in the _From Java to PeopleCode_ section of PeopleBooks.  This class holds methods that replicate
the built-in PeopleCode functions.  I am unsure if this class holds all of the available built-in functions, but I have
discovered that it holds some functions that are undocumented.

I used the [Dynamic Java in PeopleCode]( http://jjmpsj.blogspot.com/2016/07/dynamic-java-in-peoplecode.html) approach to
run the type introspection Java code via JavaScript.  Here is the JavaScript program that performs type introspection on
the `PeopleSoft.PeopleCode.Func` Java Class:

```javascript
var peopleCodeClass = new Packages.PeopleSoft.PeopleCode.Func();
var className = peopleCodeClass.getClass().getName();
var classMethods = peopleCodeClass.getClass().getMethods();

var methods = [];

for (var i = 0; i < classMethods.length; i++) {

   var method = {};
   method.name = classMethods[i].getName();
   method.returnType = classMethods[i].getReturnType().getSimpleName();

   var parameters = [];
	var paramType = classMethods[i].getParameterTypes();

	for (var j = 0; j < paramType.length; j++) {
	   var parameter = {};
	   parameter.type = paramType[j].getSimpleName();
	   parameters.push(parameter);
	}

   method.parameters = parameters;
   methods.push(method);
}

var classDetails = {};
classDetails.name = className;
classDetails.methods = methods;

var result = JSON.stringify(classDetails);
```

The program puts the function information in a JSON string and stores it in the `result` JavaScript variable.  This
program can be stored in an HTML Object (PSM_TEST in this example) and be invoked from PeopleCode as follows:

```java
Local JavaObject &joManager = CreateJavaObject("javax.script.ScriptEngineManager");
Local JavaObject &joEngine = &joManager.getEngineByName("JavaScript");
Local string &sProgram = GetHTMLText(HTML.PSM_TEST);

&joEngine.eval(&sProgram);

Local string &sResult = &joEngine.get("result").toString();

Return &sResult;
```

The `&sResult` PeopleCode variable will hold the function information represented as a JSON string.  In my 8.56.12
environment there are 1032 methods listed in the JSON output:

[0]: /assets/images/2019/10/Functions.png
[![Built-In PeopleCode Functions][0]][0]

You can also use the same approach to expose the available system variables (%UserId, etc.) that PeopleCode has to offer.
To do this, you can replace the first line in the JavaScript program to reference the `PeopleSoft.PeopleCode.SysVar` Java
Class:

```javascript
var peopleCodeClass = new Packages.PeopleSoft.PeopleCode.SysVar();
```

And the output will be similar to the Func Class output:

[1]: /assets/images/2019/10/Variables.png
[![PeopleCode System Variables][1]][1]

Keep in mind that the type introspection JavaScript program does not perform any data type mappings.  You will notice
some Java-specific parameter and return data types listed in the script’s output.
The _PeopleCode and Java Data Types Mapping_ section of PeopleBooks can be referenced to understand the corresponding
PeopleCode types.