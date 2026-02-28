---
id: 56
title: Setting Up Two-Factor Authentication in PeopleSoft (Part 1)
date: 2016-05-24T15:52:13+00:00
guid: https://www.peoplesoftmods.com/?p=56
permalink: /2fa/how-to-set-up-two-factor-authentication-in-peoplesoft/
categories:
  - Two-Factor Authentication
---
I am going to provide a tutorial on how to setup two-factor authentication (2FA) in PeopleSoft.  This is going to serve as a technical demonstration (and documentation) of how I satisfied the project requirements that were outlined in <a href="https://www.peoplesoftmods.com/uncategorized/two-factor-authentication-in-peoplesoft-part-1/" target="_blank">this </a>post.  This tutorial will be split into a few parts.

**Part 1** (_this part_) of the tutorial is to give you an overall understanding of how the 2FA process will work in your PeopleSoft environment. This part provides an overview of the necessary configuration and code changes that are needed to alter the application’s authentication process flow. Most of the information in this part of the tutorial echos what has been said in a post on _Sasank&#8217;s PeopleSoft Log_ called <a href="https://pe0ples0ft.blogspot.com/2015/04/conditional-redirect-in-signon.html" target="_blank">Conditional Redirect in SignOn PeopleCode</a>.

<!--more-->

**<a href="https://www.peoplesoftmods.com/2fa/setting-up-two-factor-authentication-in-peoplesoft-part-2/" target="_blank">Part 2</a>** of the tutorial provides an app designer project that you can plug-and-play into your environment to enable a proof-of-concept 2FA solution.

**Part 3** of the tutorial will enhance various aspects of the project that was provided in part two.

I highly suggest reading up on a couple of items in PeopleBooks before starting this tutorial.  The first item is how to <a href="http://docs.oracle.com/cd/E55243_01/pt854pbr0/eng/pt/tprt/task_ConfiguringWebProfiles-c07441.html#u76eca1ef-f7f8-437e-bec6-b1d05d2b5f41" target="_blank">configure appearance and character of the web profile</a> .  Take a good look at the _Signon Result Doc Page_ field in this section.  The second item is the <a href="http://docs.oracle.com/cd/E28394_01/pt852pbh1/eng/psbooks/tpcl/chapter.htm?File=tpcl/htm/tpcl02.htm%2337ee99c9453fb39_ef90c_10c791ddc07__3c7e" target="_blank">SetAuthenticationResult function</a>.  Familiarize yourself with the function’s parameters, especially the _ResultDocument_ parameter.

<!--more-->

In this part of the tutorial I am going to show how you can redirect users to a custom page upon successful PeopleSoft login.  This change will add an intermediate step to the delivered PeopleSoft authentication process flow.  The flow of how a user gets into the system will go from this:

**_Login Page -> Home Page_**

To this:

**_Login Page -> Custom Page -> Home Page_** 

Adding this intermediate step to the authentication process is a delivered functionality that just needs to be enabled.  There needs to be a slight configuration change to the web profile as well as some changes to the Signon PeopleCode in order for this functionality to work.

First, we need to create the custom PeopleSoft page that we want to serve as the intermediate page that will reside in between the login page and the home page.  This will be the page that users will be redirected to upon successful PeopleSoft login.  This page will be very generic for this part of the tutorial, but it will be expanded on later.  A page with some static text and a button will be all that is needed for now.

<div id="attachment_57" style="width: 626px" class="wp-caption alignnone">
  <a href="/assets/images/2016/05/PS_Page.png" rel="attachment wp-att-57"><img class="size-full wp-image-57" src="/assets/images/2016/05/PS_Page.png" alt="Custom PeopleSoft Page" width="616" height="421" srcset="/assets/images/2016/05/PS_Page.png 616w, /assets/images/2016/05/PS_Page-300x205.png 300w, /assets/images/2016/05/PS_Page-556x380.png 556w" sizes="(max-width: 616px) 100vw, 616px" /></a>
  
  <p class="wp-caption-text">
    Custom PeopleSoft Page
  </p>
</div>

The only functionality that the page needs to have for now is to be able to write a cookie to the user’s browser and redirect the user to the home page.  To achieve this, we will add some PeopleCode behind the button on the FieldEdit event.  This code will write a cookie named _PSM\_2FA\_TOKEN_ and it will redirect the users to the home page when they click the button.  We will see why we are writing this cookie when we get into the Signon PeopleCode.

<div id="attachment_58" style="width: 1180px" class="wp-caption alignnone">
  <a href="/assets/images/2016/05/Button_Code.png" rel="attachment wp-att-58"><img class="size-full wp-image-58" src="/assets/images/2016/05/Button_Code.png" alt="Field Edit PeopleCode Behind the Button" width="1170" height="270" srcset="/assets/images/2016/05/Button_Code.png 1170w, /assets/images/2016/05/Button_Code-300x69.png 300w, /assets/images/2016/05/Button_Code-768x177.png 768w, /assets/images/2016/05/Button_Code-1024x236.png 1024w" sizes="(max-width: 1170px) 100vw, 1170px" /></a>
  
  <p class="wp-caption-text">
    Button FieldEdit PeopleCode
  </p>
</div>

Once this is complete, then all that is needed is to add this page to a component and add the component to a menu.  For clarity, I named my page _PSM_2FA_, my component _PSM_2FA_, and my menu _PSM_CUSTOM_.  It is important to remember the names of these objects because they will be referenced in the Signon PeopleCode.

Now that the custom page is setup, we need to do the necessary actions to be able to redirect to this page after login.  The first thing that we need to do is make a change to the web profile.  Specifically, we need to set the _Signon Result Doc Page_ field under the _Look and Feel_ tab of the web profile to _signonresultdocredirect.html_.  Note: once this is completed you need to bounce the web server for the changes on the web profile to work.

<div id="attachment_59" style="width: 768px" class="wp-caption alignnone">
  <a href="/assets/images/2016/05/Web_Profile.png" rel="attachment wp-att-59"><img class="size-full wp-image-59" src="/assets/images/2016/05/Web_Profile.png" alt="Web Profile Configuration" width="758" height="825" srcset="/assets/images/2016/05/Web_Profile.png 758w, /assets/images/2016/05/Web_Profile-276x300.png 276w, /assets/images/2016/05/Web_Profile-349x380.png 349w" sizes="(max-width: 758px) 100vw, 758px" /></a>
  
  <p class="wp-caption-text">
    Web Profile Configuration
  </p>
</div>

In this next section of the tutorial, I am modifying existing Signon PeopleCode for demonstration purposes.  I do not recommend modifying the delivered Signon PeopleCode as i am doing here. Instead,  creating a separate Signon PeopleCode section to house our modifications is a better idea.  I outline how to do this in the second part of this tutorial.  I am going to modify the existing code for the sake of simplicity in this part.

What we need to do next is determine which Signon PeopleCode functions are being used.  To achieve this head over to the navigation: _Main Menu -> PeopleTools -> Security -> Security Objects -> Signon PeopleCode_. Make note of the functions that have the enabled checkbox checked.

<div id="attachment_60" style="width: 955px" class="wp-caption alignnone">
  <a href="/assets/images/2016/05/Functions.png" rel="attachment wp-att-60"><img class="size-full wp-image-60" src="/assets/images/2016/05/Functions.png" alt="Signon PeopleCode Functions" width="945" height="494" srcset="/assets/images/2016/05/Functions.png 945w, /assets/images/2016/05/Functions-300x157.png 300w, /assets/images/2016/05/Functions-768x401.png 768w, /assets/images/2016/05/Functions-727x380.png 727w" sizes="(max-width: 945px) 100vw, 945px" /></a>
  
  <p class="wp-caption-text">
    Signon PeopleCode Functions
  </p>
</div>

Lastly, we are going to need to modify all of the enabled Signon PeopleCode functions.  Throughout the Signon PeopleCode functions, there are going to be calls to a function named _SetAuthenticationResult_. What we need to do is modify the call to this function by changing the third parameter in the function call.  So the code should go from looking like this:

<div id="attachment_61" style="width: 524px" class="wp-caption alignnone">
  <a href="/assets/images/2016/05/Code_Before.png" rel="attachment wp-att-61"><img class="size-full wp-image-61" src="/assets/images/2016/05/Code_Before.png" alt="Delivered Function Call" width="514" height="16" srcset="/assets/images/2016/05/Code_Before.png 514w, /assets/images/2016/05/Code_Before-300x9.png 300w" sizes="(max-width: 514px) 100vw, 514px" /></a>
  
  <p class="wp-caption-text">
    Delivered Function Call
  </p>
</div>

To this:

<div id="attachment_62" style="width: 874px" class="wp-caption alignnone">
  <a href="/assets/images/2016/05/Code_After.png" rel="attachment wp-att-62"><img class="size-full wp-image-62" src="/assets/images/2016/05/Code_After.png" alt="Modified Function Call" width="864" height="154" srcset="/assets/images/2016/05/Code_After.png 864w, /assets/images/2016/05/Code_After-300x53.png 300w, /assets/images/2016/05/Code_After-768x137.png 768w" sizes="(max-width: 864px) 100vw, 864px" /></a>
  
  <p class="wp-caption-text">
    Modified Function Call
  </p>
</div>

What this code is doing is populating the _ResultDocument_ parameter and then calling the _SetAuthenticationResult_ with the populated parameter.  The _ResultDocument_ parameter either gets populated with a blank string or a string consisting of the URL to the custom page that we previously created.

When the _ResultDocument_ parameter gets populated with a blank string, the user will be taken to one of the following locations based on the given conditions:

**Login Page** if the &PSAuthResult bool is false

**Password Reset Page** if the &PSAuthResult bool is true <u>AND</u> the &EXPIRE bool is true

**Home Page** if the &PSAuthResult bool is true <u>AND</u> the &EXPIRE bool is false <u>AND</u> the user has the _PSM\_2FA\_TOKEN_

The _ResultDocument_ parameter only gets populated with the URL to the custom page when the &PSAuthResult bool is true <u>AND</u> the &EXPIRE bool is false <u>AND</u> the user doesn’t have the _PSM\_2FA\_TOKEN_.

I’d like to go into more detail of the flow of when we do the redirect process starting from the beginning of the login process.

  1. The user successfully authenticates themselves with their user id and password
  2. _SetAuthenticationResult_ function is called
  3. A _PS_TOKEN_ is issued to the user (the importance of this is explained below)
  4. The _ResultDocument_ parameter (the URL) is passed to the _signonresultdocredirect.html_ page
  5. _signonresultdocredirect.html_ redirects the user to provided URL of the custom page
  6. The user clicks the button on the custom page
  7. Custom page assigns the _PSM\_2FA\_TOKEN_ to the user
  8. Custom page redirects the user to the homepage

When the custom page redirects the user to the homepage the Signon PeopleCode is fired again.  This time however, the user will have the _PSM\_2FA\_TOKEN_ so they will be taken to the homepage.  What is going on here essentially is the _PSM\_2FA\_TOKEN_ is acting as a secondary token to the _PS_TOKEN_.  The user must have both tokens to have full access to the system.  In a later part of this tutorial we will not just give out the _PSM\_2FA\_TOKEN_ with the click of a button as we are currently doing.  Instead, we will enforce a second factor of authentication on the user before giving them the _PSM\_2FA\_TOKEN_.

Here is the code that was used in this post.

FieldEdit code behind the button:

<pre>/* Save a cookie to the user's browser and then redirect the user to the homepage */
Local object &2FA_Cookie;

&2FA_Cookie = %Response.CreateCookie("PSM_2FA_TOKEN");
&2FA_Cookie.Domain = %Request.AuthTokenDomain;
&2FA_Cookie.MaxAge = 2592000;
&2FA_Cookie.Path = "/";
&2FA_Cookie.Secure = False;
&2FA_Cookie.Value = "1";

/* Redirect to the start page Iscript. Note: this script is defined on the web profile's "Look and Feel" tab */
%Response.RedirectURL(GenerateScriptContentURL(%Portal, %Node, Record.WEBLIB_PTBR, Field.ISCRIPT1, "FieldFormula", "IScript_StartPage"));</pre>

Delivered Signon PeopleCode _SetAuthenticationResult_ function call:

<pre>SetAuthenticationResult(&PSAuthResult, &USERID, "", &EXPIRE, &daysleft);</pre>

Modified Signon PeopleCode _SetAuthenticationResult_ function call:

<pre>/* Begin 2FA Mod */
If (%PSAuthResult And
%Request.GetCookieValue("PSM_2FA_TOKEN") = "") Then
&URL = GenerateComponentContentURL("EMPLOYEE", "SA", MenuName.PSM_CUSTOM, "GBL", Component.PSM_2FA, "PSM_2FA", "");
Else
&URL = "";
End-If;
SetAuthenticationResult(&PSAuthResult, &USERID, &URL, &EXPIRE, &daysleft);
/* End 2FA Mod */</pre>