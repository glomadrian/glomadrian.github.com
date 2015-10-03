---
layout: post
title:  "Multiple data sources and caching"
date:   2015-09-29
---

Before you star reading this post, I assume you have knowledge about:

* Clean Architecture
* Repository Pattern
* Model View Presenter Pattern
* Dependency injection
* SOLID

### Why this post

<p> Is all about data, data is the base of the applications, everyday we have to
 deal with data in our apps, as developer you have to take care about how obtain
 your data and where store it, because... are you storing data, isn't it?</p>

 <p> In the most of Android applications there are an API that provide data,
 the application start, fetch the data and show it in the screen, that's perfect,
 but what happens when network connection fails? no data to fetch, no data to show,
 the application comes unusable, an awesome and sad image is showing at best.</p>

<img src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/sad_cloud_google.png' | prepend: site.baseurl }}" alt="no network image">

Maybe the connection is never lost, but, is it necessary to fetch the data from
the API every time the user enter in the application? The answer is no.

If you are an Android Developer then I suppose that you are familiar with Picasso
 library from square. Picasso takes care about obtain and store the images for
 you, if an image is already downloaded you don't need to download it again,
 why not use this approach with the data?


In this post I going to try explain how to accomplish this, always with a SOLID,
Clean and reusable code.

### Data, data and mode data


[<img src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/data_access_cache.jpg' | prepend: site.baseurl }}" alt="data sources and cache">][1]

As you can see in the picture there are two main layers on it, the domain layer
and the data access layer, this two layers must be decoupled each other.

#### Domain layer

[<img src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/cleanArchitecture.jpg' | prepend: site.baseurl }}" alt="data sources and cache">][2]

The domain layers are bussiness logic of your application, correspond with the
yellow, red and green layers on clean architecture.

#### Data Access layer

The data access layer correspond with API implementations, databases, or any source of data you like. This implementations know nothing of domain models, and can be used in other projects without any modifications, there are no part of your application logic.

### Example Application

The example application (source code link below) shows 20 Android news from today. By default if there are not data to fetch from the cloud, the news are taken from database. The update from cloud may be forced. Every time the application fetchs data from the cloud, the database should be updated.

<img  height="500" src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/appdemo.gif' | prepend: site.baseurl }}" alt="Application Demo">

### Tell me a history

This is the story of a use case, his name is **GetTodayNewsInteractor**. GetTodayNewsInteractor travels through the Presenter -> Use Case -> Repository -> Policity -> Memory and API and then returned to the View with a lot of news to show, but, what happens in this travel?

I assume you know MVP pattern. Let start form the presenter.

#### Inside the Presenter

The presenter has an injected instance of GetTodayNewsInteractor use case, that will be executed and run in a new thread.


#### Inside the GetTodayNewsInteractor

**GetAllNews** has a **NewsRepository** dependency. This repository has several ways to get the data. The interactor asks for the data and tells which policy use.


{% highlight java %}
List<NewItem> newItems = newsRepository
                        .withPolicy(NewsPolicies.DATABASE_FIRST)
                        .getTodayNews();
{% endhighlight %}

##### TIPS
 * You can have one use case per policy. For instance, GetTodayNewsCloudInteractor to force cloud update at some time.



#### Inside the NewsRepository

The repository has the responsibility to abstract the data access from other business logic. Usually a repository has an instance of a data source to get the data from it, then map it to a domain model and return to business logic. In this pattern, the repository delegates the data access to the policies and the mapping to a data sources, but it remains an abstraction layer for bussiness logic.

The repository has one or several policies to ask for data, that depends of your needs. In this example, the repository have two policies and the policy to be used can be selected at runtime.

{% highlight java %}
@Override
public List<NewItem> getTodayNews() {
  switch (policy) {
    case DATABASE_FIRST:
      return databaseFirst();
    case NETWORK_ONLY_WITH_UPDATE:
      return networkOnlyWithUpdate();
    default:
      return databaseFirst();
  }
}
{% endhighlight %}

##### TIPS
 * switch - case is for learning purposes, a policies factory is more effective and makes repository follow the open / closed principle.
 * The policy can be injected and changed at compilation time instead of runtime.
 * The logic of the policies can be reused on other projects.

#### Inside the NewsDataBaseFirstPolicy

The policy is where all of this make sense. It have several data sources injected and makes use of them to get the data in the way you want to do.

{% highlight java %}
@Override
public List<NewItem> getTodayNews() {
  try {
    return obtainFromDataBase();
  } catch (FileDataSourceException e) {
    List<NewItem> news = obtainFromCloud();
    dataBaseDataSource.saveNews(news);
    return news;
  }
}

private List<NewItem> obtainFromDataBase() {
  return dataBaseDataSource.getToadyNews();
}

private List<NewItem> obtainFromCloud() {
  return cloudNewsDataSource.getTodayNews();
}
{% endhighlight %}

This policiy tries to obtain the data from **DataBaseSource**. If there is an exception then it tries to get the data from the **CloudDataSource** and save it in database for future usages.


##### TIPS
  * The policy makes all data decisions. Keep it clean and readable.
  * Make the policy with the reusability in mind.
  * The cache invalidation happens inside the policity.

#### Inside the DataSources

The data sources are bridges between you application domain and the data domain. The data sources know the specific data source to be used (injected), and know how to convert the specific data model to the domain model (using mappers). In the above picture there are tree data sources (interfaces) **MemoryDataSource**, **DataBaseDataSource** and **CloudDataSource**. In this example I've used them because they are the most common cases, but you can create as many datasources as you need.

Example of Cloud Data Source method

{% highlight java %}
@Override
public List<NewItem> getTodayNews() {
  AlchemyResponse alchemyResponse = alchemyApi.getNews(YESTERDAY,
     TODAY, title, genere, count);
  return alchemyResponseNewsMapper.map(alchemyResponse);
}
{% endhighlight %}

##### TIPS
  * The data source only fetches data and then transforms it.

#### Back from the travel

The data response go back to the View and is presented to the User.

### The code!

The project source code can be found on [Github][3]

### Thanks

I hope you found this post interesting and useful. Please, If you think that something is bad, or can be improved or changed, send me an email and I will gladly review it.

Thanks [@jlmd][4] by the corrections made in this post.


[1]: {{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/data_access_cache.jpg' | prepend: site.baseurl }}
[2]: {{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/cleanArchitecture.jpg' | prepend: site.baseurl }}
[3]: https://github.com/glomadrian/Multiple-data-sources-and-caching-example
[4]: https://github.com/jlmd
