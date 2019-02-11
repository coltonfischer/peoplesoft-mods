---
id: 614
title: Enabling Location-Based Security in PeopleSoft with Event Mapping
date: 2016-08-29T21:08:35+00:00
guid: https://www.peoplesoftmods.com/?p=614
permalink: /emf/enabling-location-based-security-in-peoplesoft-with-event-mapping/
categories:
  - Event Mapping
---
I wanted to share a proof-of-concept approach to creating a simple application layer firewall with event mapping in PeopleSoft.  This post is similar to my [Using the Event Mapping Framework to Enforce Two-Factor Authentication](https://www.peoplesoftmods.com/2fa/using-the-event-mapping-framework-to-enforce-two-factor-authentication/) post, but this post highlights more on the general idea of using event mapping to extend the delivered PeopleSoft security model.  What I will be showing is how you can conditionally reject requests to specific resources based on the IP address that the user is coming from.  This is a simple way to add an additional layer of security to your PeopleSoft applications with very little overhead.  This functionality will be accomplished with event mapping, which is a new feature in the 8.55 PeopleTools.

<!--more-->

Here are the steps that you will need to take to achieve this functionality:

**Step 1: Defining the Protected Resources**

You will need to determine the content references that you want to enable location-based security on.  A good example would be the administrative or non-self-service components. However for this example, I am choosing to protect the “Change My Password” content reference.

**Step 2: Defining the Trusted IP Addresses**

In this step, you will need to define a trusted IP address range (or ranges).  The trusted IP address range will be the IP addresses that a user must be on in order to access the protected resources defined in step 1.  If the user’s IP address falls outside of the trusted IP address range, then the user’s IP address will not be trusted.  In this example, I created a custom page that has a grid to store the trusted IP address ranges.

[<img class="alignnone size-full wp-image-615" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Trusted_IP.png" alt="Trusted_IP" width="736" height="196" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Trusted_IP.png 736w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Trusted_IP-300x80.png 300w" sizes="(max-width: 736px) 100vw, 736px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Trusted_IP.png)

I created a table in the database to store the information.

[<img class="alignnone size-full wp-image-616" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/TRUSTED_IP_TABLE.png" alt="TRUSTED_IP_TABLE" width="418" height="267" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/TRUSTED_IP_TABLE.png 418w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/TRUSTED_IP_TABLE-300x192.png 300w" sizes="(max-width: 418px) 100vw, 418px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/TRUSTED_IP_TABLE.png)

**Step 3: Writing Application Code**

Now I will provide you with some application class PeopleCode to enforce the additional layer of security.  I created an application package named PSM\_APP\_FIREWALL and inserted an application class named BlockIP.

[<img class="alignnone size-full wp-image-617" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/APP_PKG.png" alt="APP_PKG" width="426" height="168" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/APP_PKG.png 426w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/APP_PKG-300x118.png 300w" sizes="(max-width: 426px) 100vw, 426px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/APP_PKG.png)

The BlockIP class will need to implement the PT_RCF:ServiceInterface base class in order to be used with event mapping.  The BlockIP class code checks if a user’s IP address is within the trusted IP address ranges defined in the database table mentioned in step 2.  The code will throw an error and (optionally) redirect the user if the user’s IP address is not trusted.  Here is the code.

<pre>import PT_RCF:ServiceInterface;

class BlockIP extends PT_RCF:ServiceInterface
 method Execute();
end-class;

method Execute
 /+ Extends/implements PT_RCF:ServiceInterface.execute +/
 
 /* code to check if a user's IP address in within a given range of IP addresses */
 Local number &i, &x, &y, &z;
 Local string &myipnum, &Value1, &fromipnum, &Value2, &toipnum;
 Local string &trusted = "false";
 rem get user's IP address;
 Local string &ip = %Request.RemoteAddr;
 
 rem split the ip into 4 parts;
 Local array of string &myip = Split(&ip, ".");
 rem if the IP is in 4 parts then pad each part with up to three zeros;
 If (&myip.Len = 4) Then
 For &i = 1 To 4
 rem pad each part of the ip with zeros to make a total of 3 digits;
 &myipnum = &myipnum | Rept("0", 3 - Len(&myip [&i])) | &myip [&i];
 End-For;
 End-If;
 
 rem get the ranges of trusted IP addresses;
 Local Rowset &rs1;
 &rs1 = CreateRowset(Record.PSM_TRUSTED_IP);
 &rs1.Fill("where 1=1");
 rem loop through each IP range;
 For &x = 1 To &rs1.ActiveRowCount
 rem get the lower IP of the range first and split the ip into 4 parts;
 &Value1 = &rs1(&x).PSM_TRUSTED_IP.PSM_FROM_IP.Value;
 Local array of string &fromip = Split(&Value1, ".");
 
 rem if the IP is in 4 parts then pad each part with up to three zeros;
 If (&fromip.Len = 4) Then
 &fromipnum = "";
 For &y = 1 To 4
 rem pad each part of the ip with zeros to make a total of 3 digits;
 &fromipnum = &fromipnum | Rept("0", 3 - Len(&fromip [&y])) | &fromip [&y];
 End-For;
 End-If;
 
 rem get the higher IP of the range second and split the ip into 4 parts;
 &Value2 = &rs1(&x).PSM_TRUSTED_IP.PSM_TO_IP.Value;
 Local array of string &toip = Split(&Value2, ".");
 
 rem if the IP is in 4 parts then pad each part with up to three zeros;
 If (&toip.Len = 4) Then
 &toipnum = "";
 For &z = 1 To 4
 rem pad each part of the ip with zeros to make a total of 3 digits;
 &toipnum = &toipnum | Rept("0", 3 - Len(&toip [&z])) | &toip [&z];
 End-For;
 End-If;
 
 If (Value(&myipnum) &gt;= Value(&fromipnum) And
 Value(&myipnum) &lt;= Value(&toipnum)) Then
 &trusted = "true";
 &x = &rs1.ActiveRowCount;
 End-If;
 
 End-For;
 
 rem if the ip is not trusted then block the request;
 If (&trusted = "false") Then
 rem get url to page that explains the location-based security policy (optional);
 rem Local string &Block_Request = GenerateComponentPortalURL(%Portal, %Node, MenuName.PSM_APP_FIREWALL, "GBL", Component.PSM_BLOCK, "PSM_BLOCK", "");
 rem redirect to the url (optional);
 rem %Response.RedirectURL(&Block_Request);
 rem use error to stop processing. this will prevent data being communicated to client;
 Error MsgGet(40, 20, "You are not authorized to access this component.");
 End-If;
 
end-method;</pre>

**Step 4: Defining a Related Content Service**

Once the application class is written, you will need to tie the application code to a service that will be used to map the code to component events.  To create a service you will need to navigate to PeopleTools -> Portal -> Related Content Service -> Define Related Content Service.  Here you will add a new service ID.  Be sure to select “Application Class” as the URL type and specify the package name, path, and class name.

[<img class="alignnone size-full wp-image-618" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Service.png" alt="Service" width="837" height="621" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Service.png 837w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Service-300x223.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Service-768x570.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Service-512x380.png 512w" sizes="(max-width: 837px) 100vw, 837px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Service.png)

**Step 5: Mapping the Event**

Last you will need to map the newly created service to the content references that you want to protect with location based security.  To do this, navigate to PeopleTools -> Portal -> Related Content Service -> Manage Related Content Service.  Here you will need to select the “Event Mapping” tab and then click the “Map the event of the Application pages” link.

[<img class="alignnone size-full wp-image-619" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Event_Mapping.png" alt="Event_Mapping" width="740" height="236" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Event_Mapping.png 740w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Event_Mapping-300x96.png 300w" sizes="(max-width: 740px) 100vw, 740px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Event_Mapping.png)

This is where you will select the content reference that you want to protect.  As I previously said, I will be mapping my code to the “Change My Password” content reference.

[<img class="alignnone size-full wp-image-620" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/CREF.png" alt="CREF" width="324" height="629" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/CREF.png 324w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/CREF-155x300.png 155w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/CREF-196x380.png 196w" sizes="(max-width: 324px) 100vw, 324px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/CREF.png)

The previously created service will need to be mapped to the component level “Pre Build” event with a “Pre Process” processing sequence.

[<img class="alignnone size-full wp-image-621" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/MAP_EVENT.png" alt="MAP_EVENT" width="899" height="517" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/MAP_EVENT.png 899w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/MAP_EVENT-300x173.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/MAP_EVENT-768x442.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/MAP_EVENT-661x380.png 661w" sizes="(max-width: 899px) 100vw, 899px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/MAP_EVENT.png)

This step will need to be performed for all of the content references that you want to map the service to.  Important Note:  If you are experiencing inconsistent behavior with your mapped code not firing, then take a look at this MOS document: INCONSISTENT BEHAVIOR OF EVENT MAPPING (Doc ID 2171391.1).

**Step 6: Test**

After performing the previous steps, when I attempt to access the “Change My Password” content reference from an untrusted IP address, I get the following message.[<img class="alignnone size-full wp-image-622" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Unauthorized.png" alt="Unauthorized" width="959" height="448" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Unauthorized.png 959w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Unauthorized-300x140.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Unauthorized-768x359.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Unauthorized-813x380.png 813w" sizes="(max-width: 959px) 100vw, 959px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/Unauthorized.png)

Side Note: At this point, Oracle does not offer the ability to map events to the SearchInit event of the search record. Hopefully Oracle can provide this functionality in the future.  Having this ability would prevent having to wait until after the search for the custom event to fire.

For a better user experience, I suggest redirecting the user to a separate page along with throwing the error in the BlockIP class.  For this example, I redirect the users with untrusted IP addresses to page with some static text.

[<img class="alignnone size-full wp-image-623" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/UNTRUSTED_IP.png" alt="UNTRUSTED_IP" width="855" height="354" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/08/UNTRUSTED_IP.png 855w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/UNTRUSTED_IP-300x124.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/08/UNTRUSTED_IP-768x318.png 768w" sizes="(max-width: 855px) 100vw, 855px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/08/UNTRUSTED_IP.png)

If you are interested in removing the Navigator menu item references to the protected resources that you have defined, then i suggest checking out this post on [Location-Based Menu Pruning](https://www.peoplesoftmods.com/emf/location-based-menu-pruning/).  This will help enhance the user experience, because the users will not be presented with menu items to the resources that you are protecting with location-based security.

* * *

The event mapping framework has brought the ability to create an easy to configure, bolt-on application layer firewall that would’ve been infeasible to implement in previous PeopleTools versions. This same idea of conditionally rejecting requests to resources can be applied to other conditions such as time of day, request cookie values, request header values, etc.  If you have any questions, comments, or ideas of ways to leverage event mapping, then feel free to comment below.