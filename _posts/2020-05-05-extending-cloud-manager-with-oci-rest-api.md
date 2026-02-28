---
id: 1354
title: Extending Cloud Manager with OCI REST API
date: 2020-05-06T02:33:08+00:00
guid: http://www.peoplesoftmods.com/?p=1354
permalink: /cm/extending-cloud-manager/
tags:
  - Cloud
  - REST
  - Java
categories:
  - Cloud Manager
---

Cloud Manager is a powerful tool for managing PeopleSoft Environments running on Oracle Cloud Infrastructure (OCI). There are many great features in the latest release and I highly recommend listening to the [Cloud Manager 10 Review]( https://psadmin.io/2020/01/31/222-cloud-manager-10-review/) PeopleSoft Administrator Podcast episode to understand the features of the current Cloud Manager release. As mentioned by [Graham Smith](https://i-like-trains.blogspot.com/) in the episode, the “Stop” environment feature does not stop the underlying virtual machine instance running in OCI. This is undesirable because you will still get charged for compute cycles even when the environment is “Stopped”. In this post I will discuss how the OCI REST API can be used to extend Cloud Manager to support functionality such as stopping OCI virtual machine instances.

### The OCI REST API

OCI offers a robust REST API to perform various actions to the infrastructure running on OCI. One particularly useful function is the [InstanceAction API](https://docs.cloud.oracle.com/en-us/iaas/api/#/en/iaas/20160918/Instance/InstanceAction). This endpoint allows for performing Start, Stop, Reset, Soft Stop, and Soft Reset actions on an instance running in OCI. You need to pass the OCID of the instance in the URL path and specify the action (START, STOP, etc.) to perform as a query parameter.

  

One challenging aspect of consuming the OCI REST API is the [Request Signatures](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/signingrequests.htm) requirement. You must specify a specially-crafted `Authorization` Header for each API Request. Oracle provides good examples for crafting the Request Signature in various languages and even has a Java SDK for consuming the API. However, I found that there are some Java Classes delivered with Cloud Manager that can aid in the consumption of the OCI REST API from PeopleCode without the need to introduce additional server dependencies.

### Cloud Manager Java Classes

If you take a look at the Application PeopleCode for Cloud Manager, you will notice a lot of `CreateJavaObject` calls. Cloud Manager uses Java Classes to do the heavy lifting required to support the various application features. There are a ton of Java Classes used in Cloud Manager and I will review the interesting/useful ones that can be of help in consuming the OCI REST API from PeopleCode.

### OCISettings Class

The `com.peoplesoft.pa.cl.infrastructure.oci.OCISettings` Class can be used to obtain various settings for your Cloud Manager environment.  This class holds values that are needed in determining the proper REST URL endpoint based on the deployment region as well as key information used to generate the API request signatures.  To get an idea of the properties that this class has to offer, here is some code that generates a JSON object based on the `OCISettings` class:
```java
Local JavaObject &joOciSettings = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.OCISettings");
Local JsonObject &Json = CreateJsonObject();

&Json.AddProperty("tenancyName", &joOciSettings.getTenancyName());
&Json.AddProperty("tenancyOCID", &joOciSettings.getTenancyOCID());
&Json.AddProperty("userName", &joOciSettings.getUserName());
&Json.AddProperty("userOCID", &joOciSettings.getUserOCID());
&Json.AddProperty("publicKeyFile", &joOciSettings.getPublicKeyFile());
&Json.AddProperty("privateKeyFile", &joOciSettings.getPrivateKeyFile());
&Json.AddProperty("privateKeyPassphrase", &joOciSettings.getPrivateKeyPassphrase());
&Json.AddProperty("publicKeyFingerprint", &joOciSettings.getPublicKeyFingerprint());
&Json.AddProperty("apiSigningPrivateKeyFile", &joOciSettings.getApiSigningPrivateKeyFile());
&Json.AddProperty("apiSigningPrivateKeyPassphrase", &joOciSettings.getApiSigningPrivateKeyPassphrase());
&Json.AddProperty("apiVersion", &joOciSettings.getApiVersion());
&Json.AddProperty("cmSSHPrivateKey", &joOciSettings.getCmSSHPrivateKey());
&Json.AddProperty("homeRegion", &joOciSettings.getHomeRegion());
&Json.AddProperty("deploymentRegion", &joOciSettings.getDeploymentRegion());
&Json.AddProperty("linuxImageOCID", &joOciSettings.getLinuxImageOCID());
&Json.AddProperty("windowsImageOCID", &joOciSettings.getWindowsImageOCID());
&Json.AddProperty("windowsImagePassword", &joOciSettings.getWindowsImagePassword());

Return &Json.ToString();
```

### RestEndpointsImpl Class

The `com.peoplesoft.pa.cl.infrastructure.oci.util.RestEndpointsImpl` Class can be used to generate the base URL for some of the different OCI APIs.  Here is an example of how this class can be used with the `OCISettings` class to return the base URL to the [Core Services API](https://docs.cloud.oracle.com/en-us/iaas/api/#/en/iaas/20160918/):
```java
Local JavaObject &joOciSettings = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.OCISettings");
   
Local JavaObject &joRestEndpoints = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.util.RestEndpointsImpl", &joOciSettings.getDeploymentRegion());
   
Return "https://" | &joRestEndpoints.coreBase() | "/" | &joOciSettings.getApiVersion();
```

### OCIRestServicesImpl Class

The `com.peoplesoft.pa.cl.infrastructure.oci.OCIRestServicesImpl` Class is a factory for obtaining implementations of the different OCI APIs.  An example OCI API implementation class is the `com.peoplesoft.pa.cl.infrastructure.oci.rest.OCICoreServicesImpl` Class which implements the Core Services API.  The `OCICoreServicesImpl` provides an avenue to easily consume a handful of the Core Service APIs.  Here is an example of using the `OCIRestServicesImpl` class to obtain an instance of the `OCICoreServicesImpl` class to call the [GetInstance](https://docs.cloud.oracle.com/en-us/iaas/api/#/en/iaas/20160918/Instance/GetInstance) API endpoint:
```java
Local JavaObject &joOCICoreServices = GetJavaClass("com.peoplesoft.pa.cl.infrastructure.oci.rest.OCICoreServicesImpl");
Local JavaObject &joOCIRestServices = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.OCIRestServicesImpl");
&joOCICoreServices = &joOCIRestServices.getCoreServices();
   
Local string &sOCID = "ocid1.instance.oc1.phx.abuw4ljrlsfiqw6vzzxb43vyypt4pkodawglp3wqxjqofakrwvou52gb6s5a";
Local JavaObject &joJsonObject = &joOCICoreServices.getInstance(&sOCID);
   
Return &joJsonObject.toString();
```

### OCIRestClient Class

The `com.peoplesoft.pa.cl.infrastructure.oci.OCIRestClient` Class can be used to make generic REST requests to any of the OCI APIs.  This class is useful because it provides flexibility and it performs the request signatures on all requests.  The `OCIRestClient` offers the following methods for making generic REST requests to OCI API endpoints: doGet, doPost, doPut, and doDelete.  This class offers a lot of potential and really deserves a post on its own.  I will say that I had success in using this class to perform GET requests to the OCI API, but I experienced inconsistent results when attempting to perform a POST request.  I will wait to document this class any further until I prove it to be fully functional.

### RequestSignatureImpl Class

The `com.peoplesoft.pa.cl.infrastructure.oci.RequestSignatureImpl` Class can be used to perform the request signature on an arbitrary OCI API request.  The `RequestSignatureImpl` class offers a `sign` method that takes in an `HttpRequestBase` object parameter.  The class adds the properly formatted `Authorization` header to the request for it to then be sent to an OCI API endpoint.  This class can be used in conjunction with the other Java classes mentioned above to make generic requests to the OCI API.  I really didn't want to have to resort to using this class, but the fact that I couldn't get the `OCIRestClient` to perform POST requests left me no other choice.  Here is some sample code that can be used to extend Cloud Manager to stop a VM instance using the the [InstanceAction API](https://docs.cloud.oracle.com/en-us/iaas/api/#/en/iaas/20160918/Instance/InstanceAction) endpoint:
```java
Local string &sInstanceOCID = "ocid1.instance.oc1.phx.abuw4ljrlsfiqw6vzzxb43vyypt4pkodawglp3wqxjqofakrwvou52gb6s5a";
   
Local string &sAction = "STOP";
rem Local string &sAction = "START";
   
Local JavaObject &joOciSettings = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.OCISettings");
   
Local string &sTenancyOCID = &joOciSettings.getTenancyOCID();
Local string &sUserOCID = &joOciSettings.getUserOCID();
Local string &sPublicKeyFingerprint = &joOciSettings.getPublicKeyFingerprint();
Local string &sApiSigningPrivateKeyFile = &joOciSettings.getApiSigningPrivateKeyFile();
Local string &sApiSigningPrivateKeyPassphrase = &joOciSettings.getApiSigningPrivateKeyPassphrase();
Local string &sDeploymentRegion = &joOciSettings.getDeploymentRegion();
Local string &sApiVersion = &joOciSettings.getApiVersion();
   
Local JavaObject &joRestEndpoints = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.util.RestEndpointsImpl", &sDeploymentRegion);
   
Local string &sRestBaseUrl = "https://" | &joRestEndpoints.coreBase() | "/" | &sApiVersion;
   
Local string &sInstanceActionUrl = &sRestBaseUrl | "/instances/" | &sInstanceOCID | "?action=" | &sAction;
   
Local JavaObject &joHttpPost = CreateJavaObject("org.apache.http.client.methods.HttpPost", &sInstanceActionUrl);
   
Local JavaObject &joRequestSigner = CreateJavaObject("com.peoplesoft.pa.cl.infrastructure.oci.RequestSignatureImpl", &sApiSigningPrivateKeyFile, &sApiSigningPrivateKeyPassphrase, &sTenancyOCID | "/" | &sUserOCID | "/" | &sPublicKeyFingerprint);
&joRequestSigner.sign(&joHttpPost);
   
&joHttpPost.removeHeader(&joHttpPost.getFirstHeader("content-length")); /* https://stackoverflow.com/questions/25182719 */
   
Local JavaObject &joHttpClientBuilder = GetJavaClass("org.apache.http.impl.client.HttpClientBuilder");
Local JavaObject &joCloseableHttpClient = &joHttpClientBuilder.create().useSystemProperties().build();
   
rem Local JavaObject &joResponse = &joCloseableHttpClient.execute(&joHttpPost); /* more than one overload matches */
   
Local JavaObject &joClass = GetJavaClass("java.lang.Class");
Local JavaObject &joClassLoader = GetJavaClass("java.lang.ClassLoader");
   
Local JavaObject &joCloseableHttpClientClass = &joClass.forName("org.apache.http.impl.client.CloseableHttpClient", False, &joClassLoader.getSystemClassLoader());
Local JavaObject &joExecuteArgTypes = CreateJavaObject("java.lang.Class[]", GetJavaClass("org.apache.http.client.methods.HttpUriRequest"));
Local JavaObject &joExecuteMethod = &joCloseableHttpClientClass.getDeclaredMethod("execute", &joExecuteArgTypes);
Local JavaObject &joExecuteArgs = CreateJavaObject("java.lang.Object[]", &joHttpPost);
   
Local JavaObject &joHttpResponseClass = &joClass.forName("org.apache.http.HttpResponse", False, &joClassLoader.getSystemClassLoader());
   
Local JavaObject &joResponse = &joHttpResponseClass.cast(&joExecuteMethod.invoke(&joCloseableHttpClient, &joExecuteArgs));
   
Return String(&joResponse.getStatusLine().getStatusCode());
```

### Closing Thoughts

I only touched on a small aspect of the OCI REST API in this post, but the API appears to offer complete control over your cloud infrastructure. Extending Cloud Manager with the OCI REST API offers organizations the ability to use a familiar technology (PeopleTools) to write custom automation tools to manage any infrastructure running on OCI. I think this is great because it will allow PeopleSoft developers to continue to leverage their current skillset as well as provide a consolidated system (UI) for managing cloud infrastructure.