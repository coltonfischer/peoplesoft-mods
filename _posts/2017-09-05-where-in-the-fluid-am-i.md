---
id: 1113
title: Where in the Fluid Am I?
date: 2017-09-05T17:29:47+00:00
guid: https://www.peoplesoftmods.com/?p=1113
permalink: /tips-and-tricks/where-in-the-fluid-am-i/
categories:
  - Tips and Tricks
---
When navigating in a PeopleSoft system that uses Fluid Navigation, it can be easy to lose your bearings in terms of where you actually are in the Portal Registry.  Knowing the exact Content Reference (and how to get to it) in Structure and Content is sometimes crucial when troubleshooting issues.  The problem is that the new Fluid Navigation does not directly correlate to the structure of the Portal Registry like it used to in Classic (breadcrumb) Navigation.  This results in there being no easy way to determine where a given Fluid page is in Structure and Content.  I have recently found that using the combination of a simple Bookmarklet and IScript to be sufficient enough to reveal the Portal Registry path for the pages that I navigate to.  In this post, I will share the implementation details of this helpful utility.

<!--more-->

[<span style="text-decoration: underline;"><strong>CLICK HERE</strong></span>](https://www.peoplesoftmods.com/Development/PSM_YAH.zip) to download the app designer project.  Unzip the project from the downloaded file and import the project from file in App Designer.

There is a Permission List in the project named PSM\_YAH.  This Permission List has access to the IScript that provides the Portal Registry path for a given page.  Assign the PSM\_YAH Permission List to a Role that you want to have this functionality enabled for.

[<img class="alignnone size-full wp-image-1114" src="/assets/images/2017/09/Assign-Permission-List.png" alt="Assign Permission List" width="929" height="541" srcset="/assets/images/2017/09/Assign-Permission-List.png 929w, /assets/images/2017/09/Assign-Permission-List-300x175.png 300w, /assets/images/2017/09/Assign-Permission-List-768x447.png 768w, /assets/images/2017/09/Assign-Permission-List-653x380.png 653w" sizes="(max-width: 929px) 100vw, 929px" />](/assets/images/2017/09/Assign-Permission-List.png)

Now you should be able to invoke the IScript without receiving an authorization error.  You can call the IScript by pasting in the following value into your web browser after authenticating into the application.

<pre>&lt;domain&gt;/psc/ps/&lt;portal&gt;/&lt;node&gt;/s/WEBLIB_PSM_YAH.ISCRIPT1.FieldFormula.IScript_YouAreHere</pre>

You should receive a response that states “Current Page Information Not Provided”.

To test the IScript with actual values, you can provide the menu and component URL query parameters to fetch the Portal Registry path for the values.  Below is an example call for the User Profiles Portal Registry path.

<pre>&lt;domain&gt;/psc/ps/&lt;portal&gt;/&lt;node&gt;/s/WEBLIB_PSM_YAH.ISCRIPT1.FieldFormula.IScript_YouAreHere?menu=MAINTAIN_SECURITY&component=USERMAINT</pre>

The script’s response in this case should be “Root > PeopleTools > Security > User Profiles > User Profiles”.

Alternatively, you can provide the url to a given Content Reference as a query parameter and the script will respond with the Portal Registry path to the Content Reference. Here is an example to get the Portal Registry path for the “User Profiles” Content Reference via its URL:

<pre>&lt;domain&gt;/psc/ps/&lt;portal&gt;/&lt;node&gt;/s/WEBLIB_PSM_YAH.ISCRIPT1.FieldFormula.IScript_YouAreHere?url=http://&lt;domain&gt;/psc/ps/&lt;portal&gt;/&lt;node&gt;/c/MAINTAIN_SECURITY.USERMAINT.GBL</pre>

The functionality provided by the IScript is helpful, but the call to the IScript needs to be made more functional.  To achieve this, I took some pointers from <a href="http://jjmpsj.blogspot.com/" target="_blank">Jim Marion’s</a> blog posts on bookmarklets and PeopleSoft JavaScript development techniques, and created a “You Are Here” bookmarklet.  I think exposing the Portal Registry path information for a PeopleSoft page through a bookmarklet provides an acceptable level of usability.  To make use of the bookmarklet, drag the link below into your browser’s bookmark bar.

**[PS You Are Here](javascript:(function()%7Bvar%20xhttp%20%3D%20new%20XMLHttpRequest()%3B%20xhttp.onreadystatechange%20%3D%20function()%20%7B%20if%20(this.readyState%20%3D%3D%204%20%26%26%20this.status%20%3D%3D%20200)%20%7B%20var%20yah%20%3D%20(doc.getElementById('youarehere')%20%3F%20doc.getElementById('youarehere')%20%3A%20doc.createElement('div'))%3B%20yah.id%20%3D%20'youarehere'%3B%20yah.style%20%3D%20'text-align%3A%20center%3B%20border%3Asolid%20black%201px%3B'%3B%20yah.innerHTML%20%3D%20this.responseText%3B%20var%20bodyEl%20%3D%20doc.getElementsByTagName(%22BODY%22)%5B0%5D%3B%20bodyEl.insertBefore(yah%2C%20bodyEl.firstChild)%3B%20%7D%20%7D%3B%20var%20currUrl%20%3D%20(!!frames%5B%22TargetContent%22%5D%20%3F%20!!frames%5B%22TargetContent%22%5D.strCurrUrl%20%3F%20frames%5B%22TargetContent%22%5D.strCurrUrl%20%3A%20window.location.href%20%3A%20window.location.href)%3B%20var%20parts%20%3D%20currUrl.match(%2Fps%5Bpc%5D%5C%2F(.%2B%3F)(%3F%3A_(%5Cd%2B))*%3F%5C%2F(.%2B%3F)%5C%2F(.%2B%3F)%5C%2F%5Bchs%5D%5C%2F%2F)%3B%20var%20doc%20%3D%20(frames%5B%22TargetContent%22%5D%20%3F%20frames%5B%22TargetContent%22%5D.document%20%3A%20document)%3B%20var%20divId%20%3D%20(doc.getElementById('pt_pageinfo')%20%3F%20'pt_pageinfo'%20%3A%20parts%5B2%5D%20%3F%20'pt_pageinfo_win'%20%2B%20parts%5B2%5D%20%3A%20'pt_pageinfo_win0')%3B%20var%20pageInfo%20%3D%20doc.getElementById(divId)%3B%20var%20menu%20%3D%20(pageInfo%20%3F%20pageInfo.getAttribute('menu')%20%3A%20'')%3B%20var%20component%20%3D%20(pageInfo%20%3F%20pageInfo.getAttribute('component')%20%3A%20'')%3B%20var%20mode%20%3D%20(pageInfo%20%3F%20pageInfo.getAttribute('mode')%20%3A%20'')%3B%20var%20portalNeeded%20%3D%20(frames%5B%22TargetContent%22%5D%20%3F%20'n'%20%3A%20'y')%3B%20var%20scriptUrl%20%3D%20window.location.origin%20%2B%20%22%2Fpsc%2F%22%20%2B%20parts%5B1%5D%20%2B%20%22%2F%22%20%2B%20parts%5B3%5D%20%2B%20%22%2F%22%20%2B%20parts%5B4%5D%20%2B%20%22%2Fs%2FWEBLIB_PSM_YAH.ISCRIPT1.FieldFormula.IScript_YouAreHere%3Furl%3D%22%20%2B%20encodeURIComponent(currUrl)%20%2B%20%22%26menu%3D%22%20%2B%20encodeURIComponent(menu)%20%2B%20%22%26component%3D%22%20%2B%20encodeURIComponent(component)%20%2B%20%22%26p%3D%22%20%2B%20portalNeeded%3B%20xhttp.open(%22GET%22%2C%20scriptUrl%2C%20true)%3B%20xhttp.send()%7D)())**

Now when you are on a PeopleSoft page and you have access to the IScript mentioned above, you can click the bookmarklet to get a div element injected into the page that contains the Portal Registry path information for the current page.  Here is an example of when I invoke the script on the Fluid Addresses page:

[<img class="alignnone size-full wp-image-1115" src="/assets/images/2017/09/Fluid-Addresses.png" alt="Fluid Addresses" width="1079" height="800" srcset="/assets/images/2017/09/Fluid-Addresses.png 1079w, /assets/images/2017/09/Fluid-Addresses-300x222.png 300w, /assets/images/2017/09/Fluid-Addresses-768x569.png 768w, /assets/images/2017/09/Fluid-Addresses-1024x759.png 1024w, /assets/images/2017/09/Fluid-Addresses-513x380.png 513w" sizes="(max-width: 1079px) 100vw, 1079px" />](/assets/images/2017/09/Fluid-Addresses.png)

And here is the script’s output for the Fluid Homepage:

[<img class="alignnone size-full wp-image-1116" src="/assets/images/2017/09/Fluid-Home.png" alt="Fluid Home" width="1274" height="924" srcset="/assets/images/2017/09/Fluid-Home.png 1274w, /assets/images/2017/09/Fluid-Home-300x218.png 300w, /assets/images/2017/09/Fluid-Home-768x557.png 768w, /assets/images/2017/09/Fluid-Home-1024x743.png 1024w, /assets/images/2017/09/Fluid-Home-524x380.png 524w" sizes="(max-width: 1274px) 100vw, 1274px" />](/assets/images/2017/09/Fluid-Home.png)

As you can see, each breadcrumb in the outputted Portal Registry path is clickable.  Clicking the breadcrumb will take you to the corresponding Content Reference in Structure and Content.  When I click the Fluid Homepage breadcrumb, I am taken to the Structure and Content for the Fluid Homepage Content Reference.

[<img class="alignnone size-full wp-image-1117" src="/assets/images/2017/09/Fluid-Home-CREF.png" alt="Fluid Home CREF" width="920" height="748" srcset="/assets/images/2017/09/Fluid-Home-CREF.png 920w, /assets/images/2017/09/Fluid-Home-CREF-300x244.png 300w, /assets/images/2017/09/Fluid-Home-CREF-768x624.png 768w, /assets/images/2017/09/Fluid-Home-CREF-467x380.png 467w" sizes="(max-width: 920px) 100vw, 920px" />](/assets/images/2017/09/Fluid-Home-CREF.png)

The script is also capable of giving the Portal Registry path for non-Menu/Component based Content References.  For example, my [online PeopleCode editor](https://www.peoplesoftmods.com/tips-and-tricks/online-peoplecode-editor-project/) is an IScript based Content Reference.  When I invoke the bookmarkelt on this particular page, the script responds with the correct Portal Registry path.

[<img class="alignnone size-full wp-image-1118" src="/assets/images/2017/09/IScript-CREF.png" alt="IScript CREF" width="1139" height="669" srcset="/assets/images/2017/09/IScript-CREF.png 1139w, /assets/images/2017/09/IScript-CREF-300x176.png 300w, /assets/images/2017/09/IScript-CREF-768x451.png 768w, /assets/images/2017/09/IScript-CREF-1024x601.png 1024w, /assets/images/2017/09/IScript-CREF-647x380.png 647w" sizes="(max-width: 1139px) 100vw, 1139px" />](/assets/images/2017/09/IScript-CREF.png)

I will admit that there are some rough edges with this utility, but I have found it to be very useful for the most part. While this tool is helpful in determining how to get to a particular Content Reference in Structure and Content, it fails to provide the actual path that a user took to get to the given page. For example: Which homepage the user came from, which tile the user clicked on, etc. <a href="https://community.oracle.com/ideas/19018" target="_blank">Dan Iverson has an idea</a> in the My Oracle Support Community <a href="https://docs.oracle.com/cd/E52319_01/infoportal/mosc.html" target="_blank">PeopleTools idea space</a> that seems to propose this functionality. I think having this sort of tracking functionality baked into the application could be useful in troubleshooting and replicating issues.

**Code References**

Bookmarklet JavaScript:

<pre>(function() {
 var xhttp = new XMLHttpRequest();
 
 xhttp.onreadystatechange = function() {
 if (this.readyState == 4 && this.status == 200) {
 var yah = (doc.getElementById('youarehere') ? doc.getElementById('youarehere') : doc.createElement('div'));
 yah.id = 'youarehere';
 yah.style = 'text-align: center; border:solid black 1px;';
 yah.innerHTML = this.responseText;
 var bodyEl = doc.getElementsByTagName('BODY')[0];
 bodyEl.insertBefore(yah, bodyEl.firstChild);
 }
 };

 var currUrl = (!!frames['TargetContent'] ? !!frames['TargetContent'].strCurrUrl ? frames['TargetContent'].strCurrUrl : window.location.href : window.location.href);
 var parts = currUrl.match(/ps[pc]\/(.+?)(?:_(\d+))*?\/(.+?)\/(.+?)\/[chs]\//);
 var doc = (frames['TargetContent'] ? frames['TargetContent'].document : document);
 var divId = (doc.getElementById('pt_pageinfo') ? 'pt_pageinfo' : parts[2] ? 'pt_pageinfo_win' + parts[2] : 'pt_pageinfo_win0');
 var pageInfo = doc.getElementById(divId);
 var menu = (pageInfo ? pageInfo.getAttribute('menu') : '');
 var component = (pageInfo ? pageInfo.getAttribute('component') : '');
 var mode = (pageInfo ? pageInfo.getAttribute('mode') : '');
 var portalNeeded = (frames['TargetContent'] ? 'n' : 'y');
 var scriptUrl = window.location.origin + '/psc/' + parts[1] + '/' + parts[3] + '/' + parts[4] + '/s/WEBLIB_PSM_YAH.ISCRIPT1.FieldFormula.IScript_YouAreHere?url=' + encodeURIComponent(currUrl) + '&menu=' + encodeURIComponent(menu) + '&component=' + encodeURIComponent(component) + '&p=' + portalNeeded;

 xhttp.open('GET', scriptUrl, true);
 xhttp.send();
}())</pre>

IScript PeopleCode:

<pre>Declare Function PortalOpen PeopleCode FUNCLIB_PORTAL.PORTAL_GEN_FUNC FieldFormula;

Function IScript_YouAreHere()
 Local string &sUrlParam = Unencode(%Request.GetParameter("url"));
 Local string &sMenu = Unencode(%Request.GetParameter("menu"));
 Local string &sComponent = Unencode(%Request.GetParameter("component"));
 Local string &sPortalNeeded = %Request.GetParameter("p");
 
 /* If the required parameters are not provided, then output a message */
 If (None(&sMenu) Or
 None(&sComponent)) And
 None(&sUrlParam) Then
 %Response.Write("Current Page Information Not Provided");
 Return;
 End-If;
 
 Local ApiObject &portal = PortalOpen();
 Local ApiObject &sCurrCref;
 
 /* First, try to find the CREF by using the provided Menu and Component */
 If &portal.FindCRefByURL(GenerateComponentContentURL(%Portal, %Node, @("Menuname." | &sMenu), "GBL", @("Component." | &sComponent), "", "")) &lt;&gt; Null Then
 &sCurrCref = &portal.FindCRefByURL(GenerateComponentContentURL(%Portal, %Node, @("Menuname." | &sMenu), "GBL", @("Component." | &sComponent), "", ""));
 Else
 /* Second, try to find the CREF by using the provided url (including url query parameters) */
 If (&portal.FindCRefByURL(&sUrlParam) &lt;&gt; Null) Then
 &sCurrCref = &portal.FindCRefByURL(&sUrlParam);
 Else
 /* Third, try to find the CREF by using the provided url (Excluding url query parameters) */
 If (&portal.FindCRefByURL(Split(&sUrlParam, "?")[1]) &lt;&gt; Null) Then
 &sCurrCref = &portal.FindCRefByURL(Split(&sUrlParam, "?")[1]);
 Else
 /* If all three attempts of getting the current CREF fail, then output a message */
 %Response.Write("No Content Reference Exists for the Provided Page Information (URL = " | &sUrlParam | ", " | "Menu = " | &sMenu | ", " | "Component = " | &sComponent | ")");
 Return;
 End-If;
 End-If;
 End-If;
 
 /* Check if portal wrapper is needed */
 Local string &sSCCrefLinkBase;
 Local string &sSCFldrLinkBase;
 If &sPortalNeeded = "y" Then
 &sSCCrefLinkBase = GenerateComponentPortalURL(%Portal, %Node, MenuName.PORTAL_ADMIN, "GBL", Component.PORTAL_CREF_ADM, "", "");
 &sSCFldrLinkBase = GenerateComponentPortalURL(%Portal, %Node, MenuName.PORTAL_ADMIN, "GBL", Component.PORTAL_OBJ_LIST, "", "");
 Else
 &sSCCrefLinkBase = GenerateComponentContentURL(%Portal, %Node, MenuName.PORTAL_ADMIN, "GBL", Component.PORTAL_CREF_ADM, "", "");
 &sSCFldrLinkBase = GenerateComponentContentURL(%Portal, %Node, MenuName.PORTAL_ADMIN, "GBL", Component.PORTAL_OBJ_LIST, "", "");
 End-If;
 
 /* Get the current CREF's parent folder */
 Local ApiObject &sParentFolder = &portal.FindFolderByName(&sCurrCref.ParentName);
 
 /* Get the link to the CREF in Structure and Content */
 Local string &sSCLink = &sSCCrefLinkBase | "?PORTALPARAM_PNAME=" | &sParentFolder.Name | "&PORTALPARAM_CNAME=" | &sCurrCref.Name;
 Local string &sYouAreHere = "&lt;a href=" | &sSCLink | "&gt;" | &sCurrCref.Label | "&lt;/a&gt;";
 
 While (&sParentFolder &lt;&gt; Null)
 /* Get a link to the parent folder in Structure and Content */
 &sSCLink = &sSCFldrLinkBase | "?PORTAL_NAME=" | %Portal | "&PORTALPARAM_FNAME=" | &sParentFolder.Name;
 &sYouAreHere = "&lt;a href=" | &sSCLink | "&gt;" | &sParentFolder.Label | "&lt;/a&gt;" | " &gt; " | &sYouAreHere;
 /* Get the parent folder */
 &sParentFolder = &portal.FindFolderByName(&sParentFolder.ParentName);
 End-While;
 
 %Response.Write(&sYouAreHere);
 
 &portal.close();
 
End-Function;</pre>