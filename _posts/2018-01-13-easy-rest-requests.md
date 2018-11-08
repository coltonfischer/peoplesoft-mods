---
id: 1247
title: Easy REST Requests
date: 2018-01-13T08:52:08+00:00
guid: http://www.peoplesoftmods.com/?p=1247
permalink: /tips-and-tricks/easy-rest-requests/
categories:
  - Tips and Tricks
---
One of the biggest pain points with using Integration Broker to consume third-party REST web services is the creation of all of the required metadata definitions.  If I want to perform a simple REST request to a third-party URL, then I am stuck having to create Message, Service, Service Operation, and Routing definitions.  Sometimes I just want the ability to test an API without having to create all of these definitions.  It turns out that there are a couple of delivered methods within the %IntBroker class that allow developers to code the consumption of a REST API without the need to create all of the metadata definitions listed above.  The two methods that I will be discussing are the ProvideRestTest and ConnectorRequest methods.

<!--more-->

**The ProvideRestTest Method**

Delivered usage of the ProvideRestTest method can be found behind the Integration Broker Service Operation Tester Utility.  This utility is located under Main Menu > PeopleTools > Integration Broker > Service Utilities > Service Operation Tester.  It appears that this method was created to ease the process for this utility to make generic REST requests.

Although undocumented, the ProvideRestTest method is included in the %IntBroker class and can be used within a PeopleCode program.  Below is a sample of code that makes use of this method to send a simple POST request to a third-party URL:

<pre><span style="color: #008000;">/* Get the request message from a generic REST-based Consumer Service Operation */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">Message</span> &mRequest = <span style="color: #0000ff;">CreateMessage</span>(<span style="color: #0000ff;">Operation</span>.IB_GENERIC_REST_POST);

<span style="color: #008000;">/* Set the third-party URL to send the request to */</span>
&mRequest.OverrideURIResource(<span style="color: #ff0000;">"http://httpbin.org/post"</span>);

<span style="color: #008000;">/* Populate the request message with data */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bRet = &mRequest.SetContentString(<span style="color: #ff0000;">"My Data"</span>);
&bRet = &mRequest.AddSegmentHeader(<span style="color: #ff0000;">"MY-CUSTOM-HEADER"</span>, <span style="color: #ff0000;">"XXX"</span>);

<span style="color: #008000;">/* Perform HTTP POST */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">Message</span> &mResponse = <span style="color: #0000ff;">%IntBroker</span>.ProvideRestTest(&mRequest, <span style="color: #ff0000;">"POST"</span>);

<span style="color: #008000;">/* Output the reponse code and response content */</span>
<span style="color: #0000ff;">MessageBox</span>(0, <span style="color: #ff0000;">""</span>, 0, 0, <span style="color: #0000ff;">String</span>(&mResponse.HTTPResponseCode));
<span style="color: #0000ff;">MessageBox</span>(0, <span style="color: #ff0000;">""</span>, 0, 0, &mResponse.GetContentString());</pre>

&nbsp;

**The ConnectorRequest Method**

The ConnectorRequest method of the %IntBroker class is the more well-known (and documented) method that allows to make easy REST requests to third-party URLs without having to do a ton of setup.  Documentation and sample usage of this method can be found in the <a href="https://docs.oracle.com/cd/E66686_01/pt855pbr1/eng/pt/tiba/task_BypassingIntegrationEnginestoSendMessages-497cf6.html" target="_blank">Bypassing Integration Engines to Send Messages</a> section of PeopleBooks as well as in the snippet below:

[<img class="alignnone size-full wp-image-1257" src="http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest.png" alt="ConnectorRequest" width="1744" height="619" srcset="http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest.png 1744w, http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest-300x106.png 300w, http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest-768x273.png 768w, http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest-1024x363.png 1024w, http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest-1071x380.png 1071w" sizes="(max-width: 1744px) 100vw, 1744px" />](http://www.peoplesoftmods.com/wp-content/uploads/2018/01/ConnectorRequest.png)

The biggest caveat that I have found to using the ConnectorRequest Method for REST requests is that the resulting response message does not contain the populated HTTPResponseCode property.  This forces the developer to decipher the status of the response by parsing the body of the response message. The fact that the HTTP status code is absent from responses, makes the ConnectorRequest method not very friendly for working with REST responses.  Other than this shortcoming, I believe the ConnectorRequest method is great for making easy requests.

**Important details**

One important piece of information to note is requests that get sent with the ProvideRestTest and ConnectorRequest methods will not appear in the Synchronous Services Operation Monitor.  I believe this is due to these methods bypassing the Integration Engine when sending the request messages.  This appears to be true regardless of the specified value for the Log Detail field of the corresponding Service Operation Routing.

Another important detail that often goes overlooked when consuming secure (https) third-party REST APIs is that the third-party SSL certificates must be added to the PeopleSoft keystore in order to send request messages to the third-party URL.

**Closing Thoughts**

Both of these methods are suitable for making easy REST requests in PeopleCode.  As we saw in this post however, the ProvideRestTest method is undocumented (unsupported) and the ConnectorRequest method has a vital shortcoming that makes it less appealing to be used for making REST requests.  Hopefully we will see delivered support for the ProvideRestTest method and/or additions to the ConnectorRequest method in future PeopleTools releases.