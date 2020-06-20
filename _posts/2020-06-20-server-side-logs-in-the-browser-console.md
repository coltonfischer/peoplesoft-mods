---
id: 1357
title: Server Side Logs in the Browser Console
date: 2020-06-20T16:09:03+00:00
guid: http://www.peoplesoftmods.com/?p=1357
permalink: /utilities/ps-chrome-logger/
tags:
  - Logging
  - JSON
categories:
  - Utilities
---

I did a post last year on the topic of [Fluid Component Logging](https://www.peoplesoftmods.com/tips-and-tricks/fluid-component-logging-techniques/) that demonstrated techniques to view server side log statements in the browser console. The techniques relied on using the `AddOnLoadScript` function to inject `console.log()` JavaScript statements at debug time. I have come to find that the ability to view log statements in the browser console offers a more productive way to debug PeopleSoft applications versus using things like `MessageBox` or file-based logging. The problem with the techniques that I demonstrated is that they are limited to only working on Fluid Components and you have to write a bit of boilerplate code to log anything other than simple data types. I have written a couple of libraries that address both of these issues that can allow for an easy to use logging solution.

## Console Logging with PS-Chrome-Logger

I went down several paths in search for an intuitive way to be able to globally write server side log statements that show up in the browser console.  I came up with working prototypes that involved using some interesting technology such as servlet filters and websockets.  While these prototypes worked, they felt a bit overly complicated.

I ended up settling on a solution that requires a browser extension called [Chrome Logger](https://chrome.google.com/webstore/detail/chrome-logger/noaneddfkdjfnfdakjjmocngnfkfehhd?hl=en).  I am not exactly thrilled on the idea of requiring the use of a specific browser and extension to achieve global console logging in PeopleSoft applications, but it works really well.  The Chrome Logger project has a [well written spec](https://craig.is/writing/chrome-logger/techspecs) that makes it easy to write a compatible server side implementation.  You can check out my PeopleCode implementation of Chrome Logger on GitHub - [PS-Chrome-Logger](https://github.com/coltonfischer/ps-chrome-logger).  

The PS-Chrome-Logger project (and browser extension) provides a way to perform console logging for any PeopleSoft application page where the `%Response` object is available.  The `Log` Method of the `PSM_CHROME_LOGGER:Console` class takes a `JsonNode` parameter that contains the data to log to the console.  Creating a `JsonNode` to represent a simple value (String, Number, Boolean, etc.) is trivial, but there is a bit of work needed for object and array structures and it is not practical to have to manually perform this serialization logic at debug time.  For example, consider a scenario where you might want to log the Rowset returned for `GetLevel0()` to understand the Component buffer contents.  This will require a lot of logic (loops, etc.) to output the Rowset data structure in a human-readable format.

In the next section I will discuss a solution to dynamically serialize PeopleCode data/object types to `JsonNode` to provide an easy way to log complex data structures.

## JSON Conversion with PS-Jsonify

I created a JSON serialization library called [PS-Jsonify](http://www.coltonfischer.com/ps-jsonify/).  This library allows you to seamlessly convert PeopleCode data/object types to the native `JsonObject`, `JsonArray`, and `JsonNode` object types.  One of the main drivers for building this library was to provide an easy way to convert dynamic data structures to JSON strings for the purpose of debugging PeopleSoft applications in the browser with PS-Chrome-Logger.

Below is an example of how PS-Jsonify can be used with PS-Chrome-Logger to log the contents of an Application Class object to the browser console. 

```java
Local PTPP_COLLECTIONS:Shortcut &oCref = create PTPP_COLLECTIONS:Shortcut(%Portal, "PT_CHANGE_PASSWORD_GBL");

/* Begin Debug */
Local PSM_JSON:Node &oJson = create PSM_JSON:Node();
&oJson.SetClass(&oCref);

Local PSM_CHROME_LOGGER:Console &oConsole = create PSM_CHROME_LOGGER:Console();
&oConsole.Log(&oJson);
/* End Debug */
```

The above example resulted in 6 lines of code (2 for the class imports) just to output a single debug statement.  There could've been some syntactical sugar added to reduce the lines of code, but there is no denying the fact that writing a debug statement in that manner is a chore in itself.

In the next section I will demonstrate how you can create an Application Class facade that allows for a log statement to be written in a single line of code.


## Creating a Logging Facade

The PS-Chrome-Logger and PS-Jsonify libraries can be consolidated into a single Application Class to provide a consumer-friendly facade that allows for log statements to be written in one line of code. Let me demonstrate.

I created an Application Package named "L" and added an Application Class named "O".  _The significance of these oddly short names will become apparent_.  

I implemented the "O" Application Class as follows:

```java
import PSM_JSON:Node;
import PSM_CHROME_LOGGER:Console;

class O
   method G(&paAny As any);
end-class;

method G
   /+ &paAny as Any +/
   
   Local PSM_JSON:Node &oJsonNode = create PSM_JSON:Node();
   &oJsonNode.SetValue(&paAny);
   
   Local PSM_CHROME_LOGGER:Console &oConsole = create PSM_CHROME_LOGGER:Console();
   
   &oConsole.Log(&oJsonNode);
   
end-method;
```

This class combines the JSON serialization logic with the Chrome Logger statement output and exposes the functionality with a consumer method named "G" that takes an `Any` value to log.  

**Note**
The [SetValue](http://www.coltonfischer.com/ps-jsonify/JSON%20Node/Methods/SetValue.html) method is used in the logging facade.  This method takes an `Any` value and is capable of converting all conventional data types.  It's support for object data type conversion is limited to the [object types listed here](http://www.coltonfischer.com/ps-jsonify/JSON%20Node/).
{: .notice--warning}

The above class can be used to output a log statement with the following single line of code:

```java
(CreateObject("L:O")).G("Hello from the server"); /* @debug */
```

This above statement is certainly not easy on the eyes, but there is a method to the madness.  The `CreateObject` method is used to dynamically create the logging facade class so that I am not burdened with having to import the class anytime that I want to log something.  The single character package/class/method names are not necessary, but they do allow for minimal typing when writing the log statement.  The practice of using the commented `@debug` annotation at the end of the log statements can assist in post debugging cleanup.  
