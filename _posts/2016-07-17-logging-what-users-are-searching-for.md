---
id: 386
title: Logging What Users Are Searching For
date: 2016-07-17T19:23:26+00:00
guid: http://www.peoplesoftmods.com/?p=386
permalink: /emf/logging-what-users-are-searching-for/
categories:
  - Event Mapping
  - Logging
---
Keeping a record of the transactions that are occurring in your PeopleSoft applications is a great way to prepare yourself for the inevitable security investigations that will need to occur after a security-related incident. While most logging can be done at the database level with triggers and audit tables, there are still some transactions that the database is not capable of capturing. One example of this is a transaction where a user inputs data into a search record on a component. I would say that logging this information is very important for certain components in PeopleSoft. Even though users are not altering the data in these situations, just knowing what the users are searching for can be useful information.

<!--more-->

I previously achieved this sort of logging functionality <a href="http://www.peoplesoftmods.com/servlet-filters/servlet-filters-in-peoplesoft/" target="_blank">with the use of servlet filters</a>. Logging this information with servlet filters works great, but I decided to try to achieve this functionality solely within the application with the use of the event mapping framework provided by Oracle in the 8.55 PeopleTools. I am very impressed with how easy it is to implement this sort functionality within the application with this framework.

Below is a video demonstration of the event mapping framework and how you can use it log what users are searching for. If you are interested in this functionality, then [<span style="text-decoration: underline;"><strong>CLICK HERE</strong></span>](http://www.peoplesoftmods.com/Development/PSM_LOG_SEARCH_POC.zip) to download the app designer project. Import this project into app designer and follow along in the video to see how to map the app class code to components across the system. Note: The code in the project assumes that you are on a Windows machine with a C:\temp directory.  This is where the logs are written to.  A minor change will will be needed if your machine does not have this directory.