---
id: 1247
title: Easy REST Requests
date: 2018-01-13T08:52:08+00:00
guid: http://www.peoplesoftmods.com/?p=1247
permalink: /tips-and-tricks/easy-rest-requests/
tags:
  - Web Services
  - Integration Broker
  - REST
categories:
  - Tips and Tricks
---

One of the biggest pain points with using Integration Broker to consume third-party REST web services is the creation of all of the required metadata definitions. If I want to perform a simple REST 
request to a third-party URL, then I am stuck having to create Message, Service, Service Operation, and Routing definitions. Sometimes I just want the ability to test an API without having to create 
all of these definitions. It turns out that there are a couple of delivered methods within the %IntBroker class that allow developers to code the consumption of a REST API without the need to create 
all of the metadata definitions listed above. The two methods that I will be discussing are the _ProvideRestTest_ and _ConnectorRequest_ methods.

**The ProvideRestTest Method**

Delivered usage of the ProvideRestTest method can be found behind the Integration Broker Service Operation Tester Utility. This utility is located under `Main Menu > PeopleTools > Integration Broker > 
Service Utilities > Service Operation Tester`. It appears that this method was created to ease the process for this utility to make generic REST requests.

Although undocumented, the _ProvideRestTest_ method is included in the _%IntBroker_ class and can be used within a PeopleCode program. Below is a sample of code that makes use of this method to send 
a simple POST request to a third-party URL:

```java
/* Get the request message from a generic REST-based Consumer Service Operation */
Local Message &mRequest = CreateMessage(Operation.IB_GENERIC_REST_POST);

/* Set the third-party URL to send the request to */
&mRequest.OverrideURIResource("http://httpbin.org/post");

/* Populate the request message with data */
Local boolean &bRet = &mRequest.SetContentString("My Data");
&bRet = &mRequest.AddSegmentHeader("MY-CUSTOM-HEADER", "XXX");

/* Perform HTTP POST */
Local Message &mResponse = %IntBroker.ProvideRestTest(&mRequest, "POST");

/* Output the reponse code and response content */
MessageBox(0, "", 0, 0, String(&mResponse.HTTPResponseCode));
MessageBox(0, "", 0, 0, &mResponse.GetContentString());
```

**The ConnectorRequest Method**

The _ConnectorRequest_ method of the _%IntBroker_ class is the more well-known (and documented) method that allows to make easy REST requests to third-party URLs without 
having to do a ton of setup. Documentation and sample usage of this method can be found in the 
[Bypassing Integration Engines to Send Messages](https://docs.oracle.com/cd/E66686_01/pt855pbr1/eng/pt/tiba/task_BypassingIntegrationEnginestoSendMessages-497cf6.html) 
section of PeopleBooks as well as in the snippet below (taken from PeopleBooks):

```java
Local XmlDoc &Output;
Local String &Exception;

Local Message &MSG1, &MSG2;

&MSG = CreateMessage(OPERATION.QE_FLIGHTPLAN);

&MSG.IBInfo.IBConnectorInfo.ConnectorName = "HTTPTARGET";
&MSG.IBInfo.IBConnectorInfo.ConnectorClassName = "HttpTargetConnector";

&nReturn = &MSG.IBinfo.IBConnectorInfo.AddConnectorProperties
    ("Method", "GET", %HttpProperty);
&nReturn = &MSG.IBinfo.IBConnectorInfo.AddConnectorProperties
    ("URL", "http://finance.yahoo.com/d/quotes.txt/?symbols
    =PSFT&format=l1c1d1t1", %HttpProperty);

&MSG2 = %IntBroker.ConnectorRequest(&MSG, true); // user exception property (true) passed

If &MSG2.ResponseStatus = %IB_Status_Success Then
    &Output = &MSG2.GetXmlDoc(); // get the data out of the message

Else
    &Exception = &MSG2.IBException.ToString()); // read the exception

End-If;
```

The biggest caveat that I have found to using the _ConnectorRequest_ Method for REST requests is that the resulting response message does not contain the populated 
_HTTPResponseCode_ property. This forces the developer to decipher the status of the response by parsing the body of the response message. The fact that the HTTP status 
code is absent from responses, makes the _ConnectorRequest_ method not very friendly for working with REST responses. Other than this shortcoming, I believe the 
_ConnectorRequest_ method is great for making easy requests.

**Important details**

One important piece of information to note is requests that get sent with the _ProvideRestTest_ and _ConnectorRequest_ methods will not appear in the Synchronous Services 
Operation Monitor. I believe this is due to these methods bypassing the Integration Engine when sending the request messages. This appears to be true regardless of the 
specified value for the Log Detail field of the corresponding Service Operation Routing.

Another important detail that often goes overlooked when consuming secure (https) third-party REST APIs is that the third-party SSL certificates must be added to the 
PeopleSoft keystore in order to send request messages to the third-party URL.

**Closing Thoughts**

Both of these methods are suitable for making easy REST requests in PeopleCode. As we saw in this post however, the _ProvideRestTest_ method is undocumented (unsupported) 
and the _ConnectorRequest_ method has a vital shortcoming that makes it less appealing to be used for making REST requests. Hopefully we will see delivered support for 
the _ProvideRestTest_ method and/or additions to the _ConnectorRequest_ method in future PeopleTools releases.