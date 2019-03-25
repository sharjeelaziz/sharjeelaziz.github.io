---
layout: post
title: "My total human powered miles"
excerpt: "Display human powered miles from Strava activities"
categories: articles
tags: [strava, api, raspberrypi, python, running, riding]
author: shaji
comments: true
share: true
image: human-miles.jpg
---
Login to your Strava account and select *Settings* on your Profile drop down menu. Select *My API Application* and enter details for your application. You can enter *localhost* as your Authorization Callback Domain.

![My API Application](/img/create-strava-application.jpg)

Rename *settings.ini.sample* to *settings.ini* and update *client_id* and *client_secret* values from your newly registered app.

{% highlight js %}
  [Settings]
  client_id = <client id from your app>
  client_secret = <secret from your app>
{% endhighlight %}

![Client Id and Secret](/img/strava-application.jpg)

Run *strava-update-token.py* to get the access token. When this Python script is run it should open a browser window for get permission from you. If the script is not able to open the browser, copy the URL from the terminal windows and paste it into the browser address bar manually. Once you give the permission, the script would update the settings file. You only need to run this once.

Run human-miles.py to start displaying your total miles.

Fork or download at [https://github.com/sharjeelaziz/human-miles](https://github.com/sharjeelaziz/human-miles)

<iframe width="560" height="315" src="https://www.youtube.com/embed/RryBl_NnMS4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
