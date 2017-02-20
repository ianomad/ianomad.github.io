---
layout: post
title:  "Android: The Most Popular Libraries for any decent MVP"
date:   2017-01-28 20:00
categories: android
excerpt: "This article is about the most popular
libraries I found to be super useful."
---

# Android Libraries (draft)

This article is about the most popular libraries I found to be super useful.
Most of them I learnt on Android Bootcamp organized by [Codepath](https://codepath.com/).
It's an amazing team that always stays up to date and supports Android Community by means of [Android Guides](http://guides.codepath.com/android). If you are ever interested in the best mentors for Android, you should definitely check them out.

Nevertheless, this post is about the best libraries out there that I had a chance to use and found them to be extremely convenient.

-----------------

## Table of contents

  - ButterKnife
  - Glide
  - Retrofit
  - Dagger 2
  - RxJava
  - RxLifecycle
  - Parceler
  - Icepick
  - Permissions Dispatcher

-----------------

## ButterKnife

If you have been developing for Android for at least couple of months seriously, you can't not know about [ButterKnife](http://jakewharton.github.io/butterknife/) - library developed by Jake Wharton (Rock Star in Android Open Source World).

The idea of this library is avoiding code boilerplate for injecting views into your Activities, Fragments, or pretty much anything that has a reference to a view.

Here is how your typical code would look without the library.

```java

TextView textView1;
TextView textView2;
TextView textView3;

@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);

  this.textView1 = (android.widget.TextView) findViewById(R.id.text_view_1);
  this.textView2 = (android.widget.TextView) findViewById(R.id.text_view_2);
  this.textView3 = (android.widget.TextView) findViewById(R.id.text_view_3);
}

```

With ButterKnife you can avoid all the lines of finding views manually just by annotation your views with the IDs and call ```ButterKnife.bind(this)```. The cool part is that if there is something missing, it will also throw an exception at a compile time.

```java

@BindView(R.id.text_view_1)
TextView textView1;
@BindView(R.id.text_view_2)
TextView textView2;
@BindView(R.id.text_view_3)
TextView textView3;

@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);

  ButterKnife.bind(this);
}

```

You can use bunch of other features like binding resource, events, etc., and you can use view references right after the ```ButterKnife.bind(this);``` call.

### Behind the scenes

During the compilation time, it will simply create all the boilerplate code in the same package with the bindings calling ```findViewById``` manually for you.

-----------------
