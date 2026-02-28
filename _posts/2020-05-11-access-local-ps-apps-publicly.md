---
id: 1355
title: Access Local PS Apps Publicly
date: 2020-05-12T01:53:33+00:00
guid: http://www.peoplesoftmods.com/?p=1355
permalink: /tips-and-tricks/access-local-ps-apps-publicly/
tags:
  - Cloud
  - Heroku
  - Reverse Proxy
categories:
  - Tips and Tricks
---

If you are like me and run PeopleSoft applications on your personal machine, there are times where you might want to share a feature or functionality that you have developed on the local instance to a user that is not on the LAN.  A popular solution for exposing local web apps over a public endpoint is [Ngrok](https://ngrok.com/).  I found that the free tier of Ngrok is usable for exposing a local PeopleSoft instance to the public, but it poses some limitations around the amount of connections per minute that makes it challenging to use at times.  I found several Ngrok alternatives online and one particularly interesting one was [Inlets](https://docs.inlets.dev/).  Inlets does not pose any sort of limitations, but it does require you to run your own "exit node".  In this post I will demonstrate how to host an Inlets exit node for free using Heroku that will allow for your local PeopleSoft application to be accessed publicly.

### Create Heroku Account

If you do not already have a Heroku account, then you will need to [create one here](https://signup.heroku.com/).

### Deploy to Heroku

You can automatically setup the Heroku instance of the Inlets exit node by clicking the deploy button below.

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/coltonfischer/inlets-heroku)

You will need to provide a name for the app.  The name of the app will be used in the public URL that tunnels to your localhost.  The URL format will be `<APP_NAME>.herokuapp.com`.  In this example I will name the app `ps-inlet`.

[0]: /assets/images/2020/05/Deploy.png
[![Deploy App][0]][0]

Click the Deploy app button and the deployment process will create a shared secret key that will be used to secure the tunnel link.  This key will need to be used on the host node (PeopleSoft) to be able to successfully connect.  

### Obtaining the Secret Key

After the deployment process completes, you will need to obtain the secret key that was generated during deployment.  Navigate to the settings tab of the deployed app and click the `Reveal Config Vars` button. 
 
[1]: /assets/images/2020/05/Config.png
[![Config Vars][1]][1]

Copy the value of the `TOKEN` variable to your clipboard.  This value will be used in the next step.

### Run the Inlets Client 

The Inlets project offers a client application to be invoked on the host node to be able to connect to the exit node.  This will allow for the public Heroku URL to be routed to your local PeopleSoft application. Download the client that is appropriate for your PeopleSoft application operating system.

- [Windows](https://github.com/inlets/inlets/releases/download/2.6.3/inlets.exe)
- [Linux](https://github.com/inlets/inlets/releases/tag/2.6.3) 

Run the client specifying the following parameters:

* remote - Heroku URL where your exit node is running prefixed with "wss://".  Example `wss://<APP_NAME>.herokuapp.com`.
* token - Shared secret key that was obtained the previous step.
* upstream - IP address and port number to the local PeopleSoft PIA.  Example: `http://127.0.0.1:8000`.

Here is an example command to run the client on from a Windows PeopleSoft environment:

```console
inlets client --remote wss://ps-inlet.herokuapp.com --token 67770ad403322c281e32ea340b97811ced532c4584951e0c2e20c3ef28d2879a --upstream http://127.0.0.1:8000
```

This will create the connection tunnel from the Heroku URL to your localhost.  You should see output similar to the following:

```console
2020/05/11 18:55:01 Welcome to inlets.dev! Find out more at https://github.com/inlets/inlets
2020/05/11 18:55:01 Starting client - version 2.6.3
2020/05/11 18:55:01 Upstream:  => http://127.0.0.1:8000
2020/05/11 18:55:01 Token: "67770ad403322c281e32ea340b97811ced532c4584951e0c2e20c3ef28d2879a"
time="2020-05-11T18:55:01-05:00" level=info msg="Connecting to proxy" url="wss://ps-inlet.herokuapp.com/tunnel"
```

Now you can access your local PeopleSoft instance by pointing your browser to the public Heroku app URL (`<APP_NAME>.herokuapp.com`):

[2]: /assets/images/2020/05/Public.png
[![Public PeopleSoft][2]][2]

Simply exit the Inlets client when you are ready to close the connection tunnel.
