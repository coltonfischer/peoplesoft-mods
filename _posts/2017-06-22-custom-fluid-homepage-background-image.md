---
id: 1024
title: Custom Fluid Homepage Background Image
date: 2017-06-22T17:05:17+00:00
guid: https://www.peoplesoftmods.com/?p=1024
permalink: /emf/custom-fluid-homepage-background-image/
categories:
  - Event Mapping
---
I recently got my hands on a PeopleSoft environment running the new 8.56 PeopleTools.  I have been most curious to see the advancements in the Related Content Framework Event Mapping functionality in the new Tools release.  One huge limitation with Event Mapping in the 8.55 PeopleTools was the inability to inject custom styling into Fluid pages.  The framework did not disallow this practice, but injecting custom styles into Fluid pages would generally result in the page becoming incorrectly rendered and unusable.  One particular interesting use case of injecting custom styling with Event Mapping is to change the Fluid Homepage background.  This use case was proposed by a member of the <a href="http://psadmin.io/community/" target="_blank">psadmin.io Community</a> and I had previously tried this in 8.55 and it did not work well. However, it seems to work great in the new 8.56 PeopleTools.  Below I will walk through how one can go about changing the Fluid homepage background with Event Mapping.

<!--more-->

The first thing you are going to want to do is upload the desired background image into App Designer.  This step is optional, but doing this should allow for the image to load faster when the homepage is generated. The alternative is to store the image on an external server and then reference the remote image URL in the CSS in the next step.

[<img class="alignnone size-full wp-image-1025" src="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image.png" alt="App Designer Image" width="1072" height="686" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image.png 1072w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image-300x192.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image-768x491.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image-1024x655.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image-594x380.png 594w" sizes="(max-width: 1072px) 100vw, 1072px" />](https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Image.png)

The next step is to create a custom style sheet in App Designer to set the homepage background as an image.  If you chose to upload the image into the PeopleSoft database, then you can make use of the %Image meta-html, otherwise you will have to reference the URL to the image file.  I am not the author of this CSS and I give credit and thanks to <a href="http://jjmpsj.blogspot.com/" target="_blank">Jim Marion</a> for sharing this snippet in the <a href="http://psadmin.io/community/" target="_blank">psadmin.io Community</a>.

[<img class="alignnone size-full wp-image-1026" src="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Style.png" alt="App Designer Style" width="571" height="216" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Style.png 571w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Style-300x113.png 300w" sizes="(max-width: 571px) 100vw, 571px" />](https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Designer_Style.png)

Now you will need to write the Application Class PeopleCode to add the custom style sheet to the Fluid Homepage.  We can make use of the AddStyleSheet function to inject our custom style.

[<img class="alignnone size-full wp-image-1027" src="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Class_PeopleCode.png" alt="App Class PeopleCode" width="758" height="450" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Class_PeopleCode.png 758w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Class_PeopleCode-300x178.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Class_PeopleCode-640x380.png 640w" sizes="(max-width: 758px) 100vw, 758px" />](https://www.peoplesoftmods.com/wp-content/uploads/2017/06/App_Class_PeopleCode.png)

This code will need to be mapped to the preprocessing of the component pre build event of the Fluid Homepage CREF.  If you are unfamiliar with using Event Mapping to add code to the Fluid Homepage, then you can read my post [Classic UI for Administrative Users](https://www.peoplesoftmods.com/emf/classic-ui-for-administrative-users/).  This post contains the steps needed to map Application Class PeopleCode to the Fluid Homepage CREF.

After enabling the mapping, the custom style should get injected on the Fluid Homepage and your custom background image should display.

Here is one example:

[<img class="alignnone size-full wp-image-1028" src="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1.png" alt="Custom BG 1" width="1072" height="582" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1.png 1072w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1-300x163.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1-768x417.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1-1024x556.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1-700x380.png 700w" sizes="(max-width: 1072px) 100vw, 1072px" />](https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_1.png)

And here is another:

[<img class="alignnone size-full wp-image-1029" src="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2.png" alt="Custom BG 2" width="1072" height="580" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2.png 1072w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2-300x162.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2-768x416.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2-1024x554.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2-702x380.png 702w" sizes="(max-width: 1072px) 100vw, 1072px" />](https://www.peoplesoftmods.com/wp-content/uploads/2017/06/Custom_BG_2.png)

* * *

Using Event Mapping to achieve a custom background image provides a lot of flexibility.  Since application code is used to serve the image, you have the potential to do conditional background images.  One example would be to display one image for one group of users and display a completely different image to another group of users.  You could even do something like show a particular image based on the time of day, the weather, or if it is the user’s birthday!