---
layout: post
title:  "Android: The Most Popular Libraries for any decent MVP. Part 1."
date:   2017-02-28 20:00
categories: android
excerpt: "This article is about the most popular
libraries I found to be super useful."
---

# Android Libraries - Part 1

This article is about the most popular libraries I found to be super useful.
Most of them I learnt on Android Bootcamp organized by [Codepath](https://codepath.com/).
It's an amazing team that always stays up to date and supports Android Community by means of [Android Guides](http://guides.codepath.com/android). If you are ever interested in the best mentors for Android, you should definitely check them out.

Nevertheless, this post is about the best libraries out there that I had a chance to use and found them to be extremely convenient.

-----------------

## Table of contents

  - [ButterKnife](#butterknife)
  - [Glide](#glide)
  - [Retrofit](#retrofit)

-----------------

## <a name="butterknife">ButterKnife</a>

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

  this.textView1 = (android.widget.TextView) findViewById(R.id.textView1);
  this.textView2 = (android.widget.TextView) findViewById(R.id.textView2);
  this.textView3 = (android.widget.TextView) findViewById(R.id.textView3);
}

```

With ButterKnife you can avoid all the lines of finding views manually just by annotation your views with the IDs and call ```ButterKnife.bind(this)```. The cool part is that if there is something missing, it will also throw an exception at a compile time.

```java

@BindView(R.id.textView1)
TextView textView1;
@BindView(R.id.textView2)
TextView textView2;
@BindView(R.id.textView3)
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

During the compilation time, it will simply create all the boilerplate code in the same package named ```SomeActivity_ViewBinding.java``` with the bindings calling ```findViewById``` manually for you. See example below.

```java
@UiThread
public MainActivity_ViewBinding(T target, View source) {
  this.target = target;

  target.textView1 = Utils.findRequiredViewAsType(
    source, R.id.toolbar, "field 'textView1'", TextView.class);
  target.textView2 = Utils.findRequiredViewAsType(
    source, R.id.tab_layout, "field 'textView2'", TextView.class);
  target.textView3 = Utils.findRequiredViewAsType(
    source, R.id.view_pager, "field 'textView3'", TextView.class);

  //etc...
}

@Override
@CallSuper
public void unbind() {
  T target = this.target;
  if (target == null) throw new IllegalStateException("Bindings already cleared.");

  target.toolbar = null;
  target.tabLayout = null;
  target.viewPager = null;

  //etc...
  this.target = null;
}
```

Notice that it also takes care of unlinking stuff.

-----------------

## <a name="glide">Glide</a>

Yet another awesome library that makes life so much easier. Think if you had to show images by url. Yes, in browser you simply do ```<img src='URL of your image' alt='some description' />```, while in Android it is not even close to that. To be short, you have to download the image, convert/resize/cache, and then only apply to the ```ImageView```. So much pain!

Let's see what <a href='https://github.com/bumptech/glide'>Glide does</a> for you! The very basic usage looks like below.

```java
Glide.with(context)
  .load(imageUrl)
  .into(imageView);
```

But most of the time not all phones can handle different sizes, etc., so you might wanna consider resizing the image before showing it, while Glide will cache and optimize everything for you behind the scenes!

```java
Glide.with(context)
  .load(imageUrl)
  .diskCacheStrategy(DiskCacheStrategy.SOURCE)
  .override(1024, 800)
  .centerCrop()
  .into(imageView);
```

Of course, this awesome library exposes all the possible configurations like size of cache, directories to cache to, additional utility libraries will as well help to add rounded corners and bunch of other minor but super necessary features.

## <a name="retrofit">Retrofit</a>

<a href='https://github.com/square/retrofit'>Retrofit</a> is another super useful awesome and well written library that helps you avoid writing all the boilerplate code to make HTTP requests. It's written by developers at Square.

Here are couple of code snippets that summarize the primary purpose and functionality of the library.
```java
public interface SomeApiFeedClient {

  @GET("/v1/post")
  public Call<List<Post>> getLatestPosts(@Query("category") String category);

}
```
```java
public void someMethodMakingApiCall() {
  Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("https://your.api.feed.com")
      .addConverterFactory(GsonConverterFactory.create())
      .build();

  SomeApiFeedClient apiClient = retrofit.create(SomeApiFeedClient.class);

  apiClient.getLatestPosts("cats")
      .enqueue(new Callback<List<Post>>() {
        @Override
        public void onResponse(Call<List<Post>> call, Response<List<Post>> response) {
          //response.isSuccessful(), response.message(), etc.
          //this is on the UI thread
          List<Post> posts = response.body();
        }
        @Override
        public void onFailure(Call<List<Post>> call, Throwable t) {
          Timber.e(t);
        }
      });
}
```

As you can see above, you never actually implement the HTTP call to be made. It is all handled by Retrofit. You can provide HTTP client, but it uses OkHttp by default, which is yet another library developed by Square team as well. :)
One of the nicest things is avoiding handling threads and switching back to UI thread manually. It exposes all the configurations like parsing other content types, etc.
