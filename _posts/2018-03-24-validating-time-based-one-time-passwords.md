---
id: 1364
title: Validating Time-Based One-Time Passwords
date: 2018-03-24T06:00:01+00:00
guid: https://www.peoplesoftmods.com/?p=1364
permalink: /2fa/validating-time-based-one-time-passwords/
tags:
  - QR Code
  - JavaScript
  - TOTP
categories:
  - Two-Factor Authentication
---
There are various techniques to enforce multi-factor authentication in web applications. One approach is to use Time-Based One-Time Passwords (TOTPs) as an additional authentication factor. I did a write-up on how to [implement Google Authenticator](/2fa/implementing-google-authenticator-in-peoplesoft/) to enforce this style of multi-factor authentication in PeopleSoft. In that article, I use a custom Java class on the app server to implement the TOTP validation algorithm. The problem with this approach is that it causes an undesirable app server dependency. I recently came across an article that demonstrates a [JavaScript implementation of the TOTP algorithm](https://www.thepolyglotdeveloper.com/2014/10/generate-time-based-one-time-passwords-javascript/). Since we have the ability to run JavaScript on the app server using Java’s built-in JavaScript interpreter, then we can make use of this JavaScript implementation of the TOTP algorithm to validate user submitted TOTPs. I would like to share a handy project that demonstrates this dependency-less TOTP validation technique.

This project demonstrates TOTP validation in the context of a two-factor authentication (2FA) enrollment process. This 2FA enrollment utility can be used in a self service manner to allow users to setup and enable 2FA enforcement on their accounts. If you are interesting in using this project, then perform the following installation steps.

  * [Download](/Development/PSM_2FA_ENROLLMENT.zip) and Import Project into App Designer
  * Build Project (Create Table)
  * Assign _PSM_2FA_ Permission List to User Role

Login as a privileged user and navigate to `Main Menu > PSM Projects > 2FA Enrollment`. You will be presented with the following page:

[1]: /assets/images/2018/03/2FA_Enrollment.png
[![2FA Enrollment][1]][1]

On this page, the user can configure a mobile authenticator application by [scanning a QR code](/2fa/generating-qr-codes-in-peoplesoft/). Once configured, the mobile device will generate TOTPs for the user. The user can input a generated TOTP into the input box to verify that their application is generating proper TOTPs. When the user submits the TOTP, the server validates the submitted TOTP with a JavaScript-based algorithm.

Since JavaScript is doing all of the leg work there is not really much to see PeopleCode-wise, but below is the PeopleCode that handles the TOTP validation. The real magic happens in the _PSM_SHA_ and _PSM_TOTP_ HTML Objects.

```java
Function IScript_ValidateTotp()
   
   /* Respond with error if the totp parameter is blank */
   Local string &sTotpInput = %Request.GetParameter("totp");
   If None(&sTotpInput) Then
      %Response.Write("{""status"": ""error"", ""message"": ""No Token Provided""}");
      Return;
   End-If;
   
   /* Get the user's secret ket from the DB */
   Local string &sEncryptedKey, &sKey;
   SQLExec("SELECT PSM_ENCRYPTED_KEY FROM PS_PSM_TOTP_USER WHERE PSM_USERID = :1", %UserId, &sEncryptedKey);
   
   /* Respond with error if the user does not have a secret key in the DB */
   If None(&sEncryptedKey) Then
      %Response.Write("{""status"": ""error"", ""message"": ""No Key Exists""}");
      Return;
   End-If;
   
   /* Decrypt the user's secret key */
   &sKey = Decrypt("", &sEncryptedKey);
   
   /* Use the Java ScriptEngineManager to run the JavaScript program to validate the totp value */
   Local JavaObject &manager = CreateJavaObject("javax.script.ScriptEngineManager");
   Local JavaObject &engine = &manager.getEngineByName("JavaScript");
   Local string &program = GetHTMLText(HTML.PSM_SHA) | GetHTMLText(HTML.PSM_TOTP);
   
   /* Provide the JS program with the required parameters */
   &engine.put("userKey", &sKey);
   &engine.put("userOtp", &sTotpInput);
   &engine.put("w", 5); /* Window: 30s * 5 = 2.5min */
   &engine.eval(&program);
   
   /* Respond with error if the totp value is invalid */
   Local string &result = &engine.get("result").toString();
   If &result <> "true" Then
      %Response.Write("{""status"": ""error"", ""message"": ""Invalid Token Provided""}");
      Return;
   End-If;
   
   /* Generate the url to redirect the user to */
   Local string &sUrl = GenerateScriptContentURL(%Portal, %Node, Record.WEBLIB_PTBR, Field.ISCRIPT1, "FieldFormula", "IScript_StartPage");
   
   /* Respond with success message */
   %Response.Write("{""status"": ""success"", ""message"": ""Token is Valid"", ""redirectUrl"": """ | &sUrl | """}");
   
End-Function;
```

* * *

By using Java's built-in JavaScript interpreter, we are able to run JavaScript on the app server to validate TOTPs rather than relying on a custom Java class to do this. This technique reduces server side dependencies and results in a more easily manged solution. I look forward to discovering and sharing the integration of more JavaScript-based tools into PeopleSoft using this technique.
