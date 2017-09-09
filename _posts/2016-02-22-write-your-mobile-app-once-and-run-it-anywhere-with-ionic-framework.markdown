---
layout: post
title: "Write your mobile app once and run it anywhere with Ionic Framework"
excerpt: "Hybrid mobile development with HTML5, AngularJS and Cordova"
tags: [ionic, ionicframework, mobile, javascript, angularjs, cordova, phonegap, html, css, sass]
date: 2016-02-22T08:15:08-01:00
comments: true
image:
  feature: posts/cover-ionic.png
---

I've been developing some apps using the [Ionic Framework](http://ionicframework.com/) since December 2014 and it has been interesting to watch the evolution of the framework during this time and the platform that Drifty (the company behind Ionic) has been building around it ever since. In this post I'll introduce you to this very nice tool that you can also consider for your projects.

About Ionic
-----------

Ionic is a free and open source framework that allows you to build your mobile apps with [AngularJS](https://angularjs.org/), HTML5, CSS and [Apache Cordova](https://cordova.apache.org/). It offers a library of mobile-optimized HTML, CSS, JavaScript, CSS components and tools for building higly interactive apps and it's built with [SASS](http://sass-lang.com/) and AngularJS. This is called hybrid mobile development, which is when you can develop your code once and then compile packages for more than one platform, such as Android or iOS, and they're neither native nor purely web-based, but a mix of both. Your app runs inside of a webview and uses Cordova to interface with the phone's operating system.

It's a good option if you consider that you'll have one code that will run on many platforms, so you'll need just JavaScript developers on your team, instead of specific developers for Android and specific developers for iOS. Two separate teams, two different codes, more bugs to deal with, everything twice.

Cordova
-------

![Cordova](/images/posts/cordova.jpg "Cordova")

This is the open source software used by Ionic to manage and generate apps for different kinds of platforms and to allow the Ionic app to communicate with the device. So Ionic works as a wrapper for Cordova and Cordova has many plugins to extend its functionalities. The plugins offer JavaScript interfaces to native components on your device, allowing your app to use native device capabilities beyond what is available to pure web apps. For example, there are plugins for scanning QRCode, for working with push notifications or accessing your device's camera.

ngCordova
---------

![ngCordova](/images/posts/ngcordova.png "ngCordova")

At Ionic's website, they recommend the use of [ngCordova](http://ngcordova.com/) (Angular powered by Cordova), which is a collection of [70+ AngularJS extensions](http://ngcordova.com/docs/plugins/) (wrappers) on top of the Cordova API that make it easy to build, test, and deploy Cordova mobile apps with AngularJS. 

For example, there's a plugin you can add to your project and with a few lines of code, use it to access your device's camera and retrieve a photo as a base64-encoded image. 

```
$ cordova plugin add cordova-plugin-camera
```

There's also a plugin for OAuth, another for Geolocation, SMS, etc.

Ionicons
--------

![Ionicons](/images/posts/ionicons.jpg "Ionicons")

This is the name of the icon font that comes with Ionic, also free and open source. If you want to implement a close button on your app, for example, you just have to use the class "ion-close" inside of an element:

```
<p><i class="icon ion-close"></i> Close</p>
```

Check [here](http://ionicons.com/) the list of icons they have available

Ionic CLI
---------

Ionic comes with a command line utility that makes it easy for you to do many tasks, such as start, build, run and emulate Ionic apps. 

You can use it to run a live reload server, so your app runs on your browser while you modify it from your IDE or Code Editor, such as [Brackets](http://brackets.io/) (The one I've been using and really enjoying), and your modifications appear automatically on the browser without the need to refresh the page.

```
$ ionic serve
```

You can also use this utility to [generate icon and slash screen images](http://ionicframework.com/docs/cli/icon-splashscreen.html), provide information about your runtime, upload your app to be viewed on Ionic View, among other things.

Ionic View
----------

In this [Ionic service](http://ionicframework.com/docs/cli/uploading_viewing.html) you can upload your app to their servers using the CLI utility and then view it on the Ionic View app installed on your device, Android or iOS. You have to [create an account](http://apps.ionic.io/signup) and then you can have many apps listed on your device with the option to synchronize them to new versions when you release them. It certainly saves you from the hassle of having to go through all the bureaucracy to test your apps on an Apple device, for example. It's also interesting for showing new features to your clients, who just need to have the Ionic View app on their devices.

Ionic Creator
-------------

The Ionic team created this [very useful service](http://ionic.io/products/creator) where you can make prototypes of your apps by dragging and dropping components and then export the code or the packages for each platform. As if it was not enough, they also created a mobile app for the Ionic Creator, where you can watch the changes in real time, as they get synchronized automatically and test your mobile app on your device instead of on your browser. 

In [this post](http://blog.ionic.io/announcing-the-new-ionic-creator/) from their blog you can read more about it.

Conclusion
----------

If you're looking to develop mobile apps for your company in a simple way and your apps won't need very specific resources from the native platforms, I can certainly recommend you Ionic Framework. It's fast, the code is clean and it offers everything you need for your mobile app development. It's not only a product, but a whole platform with many services being developed by Drifty and has a bright future.

You can find the source code of the [Ionic Framework on Github](https://github.com/driftyco/ionic)

