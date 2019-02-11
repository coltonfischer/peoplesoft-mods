---
id: 484
title: Defining Your Own Web Profile Custom Properties
date: 2016-07-31T09:56:36+00:00
guid: https://www.peoplesoftmods.com/?p=484
permalink: /psadmin/defining-your-own-web-profile-custom-properties/
categories:
  - PeopleSoft Administration
  - Tips and Tricks
---
I recently discovered a good piece of information provided by a member of the <a href="http://psadmin.io/community/" target="_blank">psadmin.io community</a>. They explained how they have defined their own custom properties on the web profile in PeopleSoft. I always thought that the custom property names had to be predefined under the hood by Oracle in order for you to make use of them. It turns out that this is not the case. You can define your own custom properties on the web profile so that you can use them as meta-variables anywhere on the web server. This becomes very useful for getting rid of hard-coded values that might exist on the static html pages on the web server. I will demonstrate how this can be done.

<!--more-->

For this example I have a link on my PeopleSoft login page that points to my single sign on portal.  If users want to login through the portal instead of the PeopleSoft login, then they can click this link to be taken to the portal login.

[<img class="alignnone wp-image-485 size-full" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Login_Link_Before.png" alt="PeopleSoft Login With Link" width="546" height="519" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Login_Link_Before.png 546w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Login_Link_Before-300x285.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Login_Link_Before-400x380.png 400w" sizes="(max-width: 546px) 100vw, 546px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Login_Link_Before.png)

I have the link to the portal hard-coded on this page.  Here is an example of what the source looks like on this signin.html page:

[<img class="alignnone wp-image-486 size-full" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_Before.png" alt="Source Code Before" width="795" height="150" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_Before.png 795w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_Before-300x57.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_Before-768x145.png 768w" sizes="(max-width: 795px) 100vw, 795px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_Before.png)

To get rid of the hard-coded link in the source code, we can define a custom property on the web profile that will point to the link.  To do this, navigate to _PeopleTools -> Web Profile -> Web Profile Configuration_. Select the name of your web profile and click on the &#8220;_Custom Properties_&#8221; tab. Add a custom property with a name of your choosing.  I set the property name of mine to &#8220;_mylink_&#8220;. Set the validation type to string and input the link&#8217;s value into the property value field.

[<img class="alignnone wp-image-487 size-full" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Custom_Property.png" alt="Custom Property" width="864" height="411" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Custom_Property.png 864w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Custom_Property-300x143.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Custom_Property-768x365.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Custom_Property-799x380.png 799w" sizes="(max-width: 864px) 100vw, 864px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Custom_Property.png)

Now you need to change the source code of the signin.html page to point to the newly defined property.  All that is required for this is to replace the hard-coded link with a meta-html reference to the newly defined custom property. Since I named my custom property &#8220;_mylink_&#8220;, then I will replace my hard-coded link with a <%=mylink%> meta-html tag.

[<img class="alignnone wp-image-488 size-full" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_After.png" alt="Source Code After" width="786" height="153" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_After.png 786w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_After-300x58.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_After-768x149.png 768w" sizes="(max-width: 786px) 100vw, 786px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/07/Source_Code_After.png)

A web server bounce and cache clear is required for the changes to go into effect. Once you do this, you will notice that the meta-html variable in the source code gets populated with the value that is defined in the custom property of the web profile.

It is worth noting that these custom properties are stored in the _PSWEBPROFPROP_ table in the database.  If you have links (or other variables) that are environment-specific, then the population of these values to the web profile can be done by updating this table.  This can be useful because you can populate these properties in your post-refresh database scripts.  This can ultimately lead to less maintenance and overhead by only having to maintain a single version of your static html pages.