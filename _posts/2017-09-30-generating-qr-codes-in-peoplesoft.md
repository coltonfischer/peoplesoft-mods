---
id: 1164
title: Generating QR Codes in PeopleSoft
date: 2017-09-30T14:30:15+00:00
guid: https://www.peoplesoftmods.com/?p=1164
permalink: /2fa/generating-qr-codes-in-peoplesoft/
categories:
  - Tips and Tricks
  - Two-Factor Authentication
---
The PeopleCode language is not known for natively supporting cutting edge technical functionalities.  However, it is common for the PeopleSoft Developer to be thrown a technically advanced requirement from time to time.  When this sort of occasion arises, I like to extend PeopleCode with Java.  The possibilities are practically endless when extending PeopleCode with Java.  The problem though, is that the PeopleSoft app server may not be equipped with the proper Java packages to execute the required Java functions.  While we have the ability to deploy additional Java classes to the PeopleSoft app server, this practice is not always acceptable.  A clever alternative is to use the built-in JavaScript interpreter in Java and write JavaScript to overcome the technical hurdle.   In this post, I will demonstrate how I am able to use Java’s ScriptEngineManager class to execute JavaScript to generate QR codes in PeopleSoft.

<!--more-->

QR codes are a great way to transport information into a mobile device that would otherwise be tedious to input manually.  QR codes have been widely adopted in the marketing space, but there are many other great use cases for QR codes.  One use case for QR codes is for transporting secret keys into mobile authenticator applications such as Google Authenticator or Authy.  Once the key information has been communicated, the mobile application can begin generating Time-Based One-Time Passwords (TOTPs) for the user.

For demonstration purposes, I created a simple IScript that is capable of returning a QR code for the logged in user.  The data inside of the QR code is a 16 character, base32 encoded string.  This string is used as the secret key for a mobile authenticator application to generate TOTPs for the user.

<span style="text-decoration: underline;"><strong><a href="/assets/downloads/PSM_QR_CODE.zip">CLICK HERE</a></strong></span> to download the app designer project.  Unzip the project from the downloaded file and import the project from file in App Designer.  To access the QR code generating IScript, you will need to assign the _PSM_QR_ Permission List to a Role of the users that you want to generate QR codes for.

After performing the security setup, you can login as the privileged user and invoke the IScript.  You can point your browser to the following URL to generate a QR code for the user:

<pre>&lt;domain&gt;/psc/ps/&lt;portal&gt;/&lt;node&gt;/s/WEBLIB_PSM_QR.ISCRIPT1.FieldFormula.IScript_GenQR</pre>

And you should get a QR code for the logged in user:

[<img class="alignnone size-full wp-image-1165" src="/assets/images/2017/09/QR-Code.png" alt="QR Code" width="798" height="471" srcset="/assets/images/2017/09/QR-Code.png 798w, /assets/images/2017/09/QR-Code-300x177.png 300w, /assets/images/2017/09/QR-Code-768x453.png 768w, /assets/images/2017/09/QR-Code-644x380.png 644w" sizes="(max-width: 798px) 100vw, 798px" />](/assets/images/2017/09/QR-Code.png)

This QR code can be scanned into a mobile authenticator application and it should immediately start generating TOTPs.

QR code generation is a neat functionality, but what I really want to highlight on is how simple the application PeopleCode is behind the IScript.  I start off by generating the URL that I want to create the QR code based off of.

<pre><span style="color: #008000;">/* Generate random 16 character Base32 string */</span>
<span style="color: #0000ff;">Local array</span> of <span style="color: #0000ff;">string</span> &sBase32Chars = <span style="color: #0000ff;">CreateArray</span>(<span style="color: #ff0000;">"A"</span>,<span style="color: #ff0000;"> "B"</span>,<span style="color: #ff0000;"> "C"</span>,<span style="color: #ff0000;"> "D"</span>,<span style="color: #ff0000;"> "E"</span>,<span style="color: #ff0000;"> "F"</span>,<span style="color: #ff0000;"> "G"</span>,<span style="color: #ff0000;"> "H"</span>,<span style="color: #ff0000;"> "I"</span>,<span style="color: #ff0000;"> "J"</span>,<span style="color: #ff0000;"> "K"</span>,<span style="color: #ff0000;"> "L"</span>,<span style="color: #ff0000;"> "M"</span>,<span style="color: #ff0000;"> "N"</span>,<span style="color: #ff0000;"> "O"</span>,<span style="color: #ff0000;"> "P"</span>,<span style="color: #ff0000;"> "Q"</span>,<span style="color: #ff0000;"> "R"</span>,<span style="color: #ff0000;"> "S"</span>,<span style="color: #ff0000;"> "T"</span>,<span style="color: #ff0000;"> "U"</span>,<span style="color: #ff0000;"> "V"</span>,<span style="color: #ff0000;"> "W"</span>,<span style="color: #ff0000;"> "X"</span>,<span style="color: #ff0000;"> "Y"</span>,<span style="color: #ff0000;"> "Z"</span>,<span style="color: #ff0000;"> "2"</span>,<span style="color: #ff0000;"> "3"</span>,<span style="color: #ff0000;"> "4"</span>,<span style="color: #ff0000;"> "5"</span>,<span style="color: #ff0000;"> "6"</span>,<span style="color: #ff0000;"> "7"</span>);
<span style="color: #0000ff;">Local integer</span> &i;
<span style="color: #0000ff;">Local string</span> &sKey;
<span style="color: #0000ff;">For</span> &i = 1 <span style="color: #0000ff;">To</span> 16
  &sKey = &sKey | &sBase32Chars [<span style="color: #0000ff;">Int</span>(<span style="color: #0000ff;">Rand</span>() * 32) + 1];
<span style="color: #0000ff;">End-For</span>;

<span style="color: #008000;">/* Supply the (arbitrary) domain name for the account to be associated with */</span>
<span style="color: #0000ff;">Local string</span> &sHost = <span style="color: #ff0000;">"peoplesoftmods.com"</span>;

<span style="color: #008000;">/* Generate the URL to be scanned into the authentication app */</span>
<span style="color: #0000ff;">Local string</span> &sQRUrl = <span style="color: #ff0000;">"otpauth://totp/"</span> | <span style="color: #0000ff;">%UserId</span> | <span style="color: #ff0000;">"@"</span> | &sHost | <span style="color: #ff0000;">"?secret="</span> | &sKey;</pre>

Next, I feed the generated URL into some JavaScript code that is capable of <a href="https://github.com/papnkukn/qrcode-svg" target="_blank">generating QR codes in SVG format</a>.  This JavaScript code gets executed by Java’s built-in JavaScript interpreter. After the interpreter executes the script, I am able to reference the SVG image output “qrSvg” variable using the built-in get method.

<pre><span style="color: #008000;">/* Get the QR Code SVG JS library and the JS command to generate a QR Code from a given value */</span>
<span style="color: #0000ff;">Local string</span> &sJSprogram = <span style="color: #0000ff;">GetHTMLText</span>(<span style="color: #0000ff;">HTML</span>.PSM_QRCODE_SVG) | <span style="color: #0000ff;">GetHTMLText</span>(<span style="color: #0000ff;">HTML</span>.PSM_QR_CODE, &sQRUrl);

<span style="color: #008000;">/* Use the Java ScriptEngineManager to run the JavaScript program to create the QR Code */</span>
<span style="color: #0000ff;">Local JavaObject</span> &manager = <span style="color: #0000ff;">CreateJavaObject</span>(<span style="color: #ff0000;">"javax.script.ScriptEngineManager"</span>);
<span style="color: #0000ff;">Local JavaObject</span> &engine = &manager.getEngineByName(<span style="color: #ff0000;">"JavaScript"</span>);
&engine.eval(&sJSprogram);

<span style="color: #008000;">/* Get the outputted SVG image from the JavaScript variable */</span>
<span style="color: #0000ff;">Local string</span> &sSVGImage = &engine.<span style="color: #0000ff;">get</span>(<span style="color: #ff0000;">"qrSvg"</span>).toString();</pre>

Last, I output the SVG QR code and additional details to the screen using the write method of the %Response class.

<pre><span style="color: #008000;">/* Output the SVG image and the account details */</span>
<span style="color: #0000ff;">%Response</span>.Write(<span style="color: #ff0000;">"&lt;br&gt;&lt;b&gt;Scan the QR code or enter the secret key into your authentication app&lt;/b&gt;&lt;br&gt;"</span>);
<span style="color: #0000ff;">%Response</span>.Write(&sSVGImage);
<span style="color: #0000ff;">%Response</span>.Write(<span style="color: #ff0000;">"&lt;br&gt;Account Name: "</span> | <span style="color: #0000ff;">%UserId</span> | <span style="color: #ff0000;">"@"</span> | &sHost);
<span style="color: #0000ff;">%Response</span>.Write(<span style="color: #ff0000;">"&lt;br&gt;Secret Key: "</span> | &sKey);</pre>

* * *

In a [previous post](https://www.peoplesoftmods.com/2fa/implementing-google-authenticator-in-peoplesoft/), I demonstrated how I was able to use the Google Charts API to generate QR codes in PeopleSoft.  Using the third-party API was an easy solution to the problem, but I think it is best to limit relying on third-parties as much as possible. In this post, we saw how we can utilize “delivered” techniques to generate QR Codes without creating additional dependencies.

I think using Java’s Built-in JavaScript interpreter to perform technically challenging tasks is a really cool way to both overcome the shortcomings of the PeopleCode language and provide for a lifecycle management-friendly solution.  If you are interested in reading and understanding more about this technique, then I highly suggest checking out Jim Marion’s posts <a href="http://jjmpsj.blogspot.com/2015/09/javascript-on-app-server-scripting.html" target="_blank">JavaScript on the App Server: Scripting PeopleCode</a> and <a href="http://jjmpsj.blogspot.com/2016/07/dynamic-java-in-peoplecode.html" target="_blank">Dynamic Java in PeopleCode</a>.