---
layout: post
title: "Going to production with an Angular CLI app"
date: 2017-05-04
---
 - an app that runs fine with `ng serve` may not compile 
 - ng steps
   - `ng build` compiles the app to /dist by default
     - serve this app and see if it runs.  In my case it did not.  Angular routes aren't working, and everything is getting passed to the api, with the exception of the '/' route.
