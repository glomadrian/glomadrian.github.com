---
layout: post
title:  "Multiple data sources and caching"
date:   2015-09-29
---

Before you star reading this post, i assume you have knowledge about:

* Clean Architecture
* Repository Pattern
* Model View Presenter Pattern
* Dependency injection
* SOLID

### Why this post

<p> Is all about data, data is the base of the applications, everyday we have to
 deal with data in our apps, as developer you have to take care about how obtain
 your data and where store it, because... you are storing it isn't you??</p>

 <p> In the most of Android applications there are an API that provide data,
 the application start, fetch the data and show it in the screen, thats perfect,
 but what happens when network connection fails? no data to fecth, no data to show,
 the application comes unusable, an awesome and sad image is showing at best</p>

<img src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/sad_cloud_google.png' | prepend: site.baseurl }}" alt="no network image">

Maybe the connection is never lost, but is necessary to fetch the data from the API
every time the user enter in the application?, the answer is no.

if you are an Android Developer i suppose you are familiar with the Picasso library
from square, picasso take care about obtain and store the images for you, if an image
is already downloaded you don't need to download it again, why don't use this approach with the data?


In this post i will try to explain how to accomplish this, always with a SOLID,
Clean and reusable code.

### Data, data and mode data
