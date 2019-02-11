---
id: 27
title: Two-Factor Authentication in PeopleSoft
date: 2016-02-18T05:12:26+00:00
guid: https://www.peoplesoftmods.com/?p=27
permalink: /2fa/two-factor-authentication-in-peoplesoft-part-1/
categories:
  - Two-Factor Authentication
---
<p style="text-align: justify;">
  This post is to document my two-factor authentication (2FA) project that I have implemented in PeopleSoft.  This project was done because there was a desire to add an additional layer of security to the application without having to worry about the costs associated with a vender-supplied 2FA solution.  A customization like this would seem to be somewhat infeasible to build in-house, but leveraging some of Oracle’s delivered functionality coupled with a well thought-out design, made this project&#8217;s implementation quite simple.  This is relatively speaking of course, after all, this project does add an entire new step to the delivered PeopleSoft authentication process. I am only going to talk about the requirements, specifications, and design in this post as well as provide a short demo. I am writing a tutorial <a href="https://www.peoplesoftmods.com/uncategorized/how-to-set-up-two-factor-authentication-in-peoplesoft/" target="_blank">here</a> that shows the steps on how to implement this.
</p>

<p style="text-align: justify;">
  <!--more-->
</p>

<table>
  <tr>
    <td width="186">
      <strong>Requirements</strong>
    </td>
    
    <td width="203">
      <strong>Specifications</strong>
    </td>
    
    <td width="159">
      <strong>Design</strong>
    </td>
  </tr>
  
  <tr>
    <td width="186">
      Implement a 2FA process that is imposed on users upon successful login.
    </td>
    
    <td width="203">
      After successful username/password authentication, redirect users to an intermediate (custom) 2FA page to do the 2FA process before taking them to the homepage
    </td>
    
    <td width="159">
      Modify the <em>Signon PeopleCode</em> to do the redirect.  Use the <em>ResultDocument</em> parameter of the <em>SetAuthenticationResult</em> function to redirect users to the 2FA page.
    </td>
  </tr>
  
  <tr>
    <td width="186">
      Initial implementation of this project should only enforce 2FA on admin-level users that are accessing the system from outside the enterprise network.
    </td>
    
    <td width="203">
      During sign on, the logic must conditionally redirect authenticated users to the 2FA page based on their PS roles and their IP address.
    </td>
    
    <td width="159">
      Make use of the <em>IsUserInRole</em> function and the <em>RemoteAddr</em> property of the <em>%Request</em> class to determine the user’s security access and location.
    </td>
  </tr>
  
  <tr>
    <td width="186">
      The enforcement of 2FA should be scalable in respect to user roles in PeopleSoft as well as login locations.
    </td>
    
    <td width="203">
      A custom page needs to exist in PS that will store PS roles and IP addresses that 2FA will be enforced on.  The sign on logic must reference this data to determine whether or not to challenge users for 2FA.
    </td>
    
    <td width="159">
      Create a PS page with grids to house the roles and IP addresses that 2FA will be enforced on.  Reference this data in the <em>Signon PeopleCode</em>.
    </td>
  </tr>
  
  <tr>
    <td width="186">
      Give the users the ability to receive time-sensitive codes to either their email or cell phone for the 2FA process.
    </td>
    
    <td width="203">
      On the 2FA page, prompt the user to receive a time-based one-time password (TOTP) to their primary email or cellphone that is stored in the database.  Use a third-party SMS gateway the send the TOTP to the user’s cellphone.
    </td>
    
    <td width="159">
      Write an algorithm to generate TOTPs.  Email the TOTP to the user’s primary email address using the <em>MCFOutboundEmail</em> class or send the TOTP to the user’s cellphone by consuming <em>Clickatell’s</em> SMS gateway web service thorough their RESTful API.
    </td>
  </tr>
  
  <tr>
    <td width="186">
      Provide functionality for users that frequently login from the same location to not have the do the 2FA process every time they login.
    </td>
    
    <td width="203">
      Provide an option on the 2FA page to “<em>Remember This Location”</em>.  Use a cookie-based solution to keep track of users that have previously performed 2FA from that location.  The cookie’s uniqueness should be based off of user attributes and location.
    </td>
    
    <td width="159">
      Use the <em>CreateCookie</em> method of the <em>%Response</em> class to store a cookie to the user’s bowser.  The cookie&#8217;s contents will be a hash of the login IP address along with user-specific attributes.
    </td>
  </tr>
</table>

Here is the flow of the authentication process. The start is the login page and the end is the home page.

<div id="attachment_141" style="width: 831px" class="wp-caption alignnone">
  <a href="https://www.peoplesoftmods.com/wp-content/uploads/2016/02/2FA_PeopleSoft_Flowchart-1.png" rel="attachment wp-att-141"><img class="wp-image-141 size-full" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/02/2FA_PeopleSoft_Flowchart-1.png" alt="Two-Factor Authentication in PeopleSoft Flowchart" width="821" height="630" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/02/2FA_PeopleSoft_Flowchart-1.png 821w, https://www.peoplesoftmods.com/wp-content/uploads/2016/02/2FA_PeopleSoft_Flowchart-1-300x230.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/02/2FA_PeopleSoft_Flowchart-1-768x589.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/02/2FA_PeopleSoft_Flowchart-1-495x380.png 495w" sizes="(max-width: 821px) 100vw, 821px" /></a>
  
  <p class="wp-caption-text">
    Two-Factor Authentication in PeopleSoft Flowchart
  </p>
</div>

&nbsp;

<p style="text-align: justify;">
  Here is a short video demonstration of this two-factor authentication solution for PeopleSoft.
</p>



If you are interested in the details of how to implement this project, then go [here](https://www.peoplesoftmods.com/uncategorized/how-to-set-up-two-factor-authentication-in-peoplesoft/) to view the tutorial.