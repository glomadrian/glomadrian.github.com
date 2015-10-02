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
 but what happens when network connection fails? no data to fecth, no data to show,
 the application comes unusable, an awesome and sad image is showing at best</p>

<img src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/sad_cloud_google.png' | prepend: site.baseurl }}" alt="no network image">

Maybe the connection is never lost, but it is necessary to fetch the data from the API,
every time the user enter in the application?, the answer is no.

if you are an Android Developer then I suppose you are familiar with the Picasso library
from square, picasso take care about obtain and store the images for you, if an image
is already downloaded you don't need to download it again, why not use this approach with the data?


In this post I going to try explain how to accomplish this, always with a SOLID,
Clean and reusable code.

### Data, data and mode data


[<img src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/data_access_cache.jpg' | prepend: site.baseurl }}" alt="data sources and cache">][1]

As you can see in the picture there are two main layers on it, the domain layer
and the data access layer, this two layers must be decoupled each other.

#### Domain layer

The domain layers are bussiness logic of your application, correspond with the
yelow, red and green layers on [this][2] clean architecture images.

#### Data Access layer

The data access layer correspond with API implementations, databases, or any source of data you like,
this implmentations know nothing of domain models, and can be used in other projects
without any modifications, there are no part of your application logic.


### Example Application

The example application (source code link below) show 20 Android news from today,
by default the news are take from database, if not are fetch from the Cloud. The update
from Cloud may be forced, every time the application fetch data from the cloud
the database should be updated.

<img  height="500" src="{{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/appdemo.gif' | prepend: site.baseurl }}" alt="Application Demo">

### Tell me a history

This is the story of a use case, his name is **GetTodayNewsInteractor**. GetTodayNewsInteractor travel
thought the Presenter - Use Case - Repository - Policity - Memory and API
and then returned to the View with a lot of news to show, but what happens in this travel?

I assume you know MVP pattern, lest start form the presenter

#### Inside the Presenter

The presenter have a injtected instance of GetTodayNewsInteractor use case, that will be
executed and run in a new thread

#### Inside the GetTodayNewsInteractor

**GetAllNews** have a dependency or a **NewsRepository**, this repository have several
ways to get the data, the interactor ask for the data and tells the policity to
use

{% highlight java %}
List<NewItem> newItems = newsRepository
                        .withPolicy(NewsPolicies.DATABASE_FIRST)
                        .getTodayNews();
{% endhighlight %}

##### TIPS
 * You can have one use case per policy f.i GetTodayNewsCloudInteractor to force
 cloud update at some time


#### Inside the NewsRepository

The repository have the responsibility to abstract the data access from other
business logic. Usually a repository have a instance of a data source to get
the data from it, then map it to a domain model and return to business logic, in
this pattern the repository delegates the data access to the policies and the apping to a
data sources but it reamains an abstraction layer for bussiness logic.

The Repository have one of several policies to ask for data, that depends of
your needs, in this example, the repository have two policies and the policy to
be use can be selected at runtime

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
 * switch - case is for learning purposes, a policies factory is more effective
 and make Repository follow the open / closed principle
 * The policy can be injected and changed at compilation time instead of runtime
 * The logic of the policies can be reused on other projects

#### Inside the NewsDataBaseFirstPolicy

The policy is where all of this make sense, it have several data sources injtected
and make use of them to get the data in the way you want to do.

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

 This policity try to obtain the data from **DataBaseSource**, if getting exception
 then try to get from the **CloudDataSource** and save it to data base for
 future usage.

##### TIPS
  * The policy make all data decisions, keep it clean and readable
  * Make the policy with the reusability in mind

#### Inside the DataSources

The data sources are bridges between you application domain and the data domain,
the data sources know the especific data source to be used (injected), and know how to
convert the specific data model to the domain model (using mappers), in the picture
above there are tree data sources (interfaces) **MemoryDataSource**, **DataBaseDataSource** and
**CloudDataSource**, I used these to be the most common but can be anything you want.

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
  * The data source only fetch data and transform it  

#### Back from the travel

The data response go back to the View and is presented to the User

### The code!

The project source code can be found on [Github][3]

### Thanks

I hope you found this post interesting and useful
if you think something can be improved or changed I'll be happy to send me an email


[1]: {{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/data_access_cache.jpg' | prepend: site.baseurl }}
[2]: {{ '/assets/img/2015-09-28-multiple-data-sources-and-caching/cleanArchitecture.jpg' | prepend: site.baseurl }}
[3]: https://github.com/glomadrian/Multiple-data-sources-and-caching-example
