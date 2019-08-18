---
layout: post
title:  "Android Fun Libraries. Part 2"
date:   2017-03-09 20:00
categories: android
excerpt: "This article is about the most popular
libraries I found to be super useful."
---

# Android Libraries - Part 2

This is a continuation of the previous post of about fun libraries for Android apps.

-----------------

## Table of contents

  - [Dagger 2](#dagger2)

-----------------

## <a name="dagger2">Dagger 2</a>

Android Development is a lot about lifecycle of its view and it's very crucial to have all of your dependency graph populated at the time of UI rendering. Now dependency graph can be necessary at a time when a user opens launcher activity, receives a notification and taps on it, or you might need to populate it when service needs to handle a background job, etc.

<img src="{{site.baseurl}}/assets/images/dependency-graph.png" alt="Drawing" style="width: 100%;"/>

### Why not use Dagger or Guice?

The main advantage of Dagger 2 is compile time code generation. So, instead of traditional type lookups and reflection based population of the objects, it will generate statically typed factories that will provide runtime object creations which is obviously much faster than reflection.

### Why is this important for Android?

Every time you receive a notification or open a view, you need the same dependency graph generated in a quick manner. You cannot afford cold startup which will hang the whole UI. Traditional approach of populating objects graph with a cold startup is not sufficient. If you are used to dependency injection on the backend and can afford couple of seconds delay, before your server gives your the first 200 OK for a health check, Android should respond fast!
