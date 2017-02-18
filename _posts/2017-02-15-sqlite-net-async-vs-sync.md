---
layout:     post
title:      SQLite.NET > async VS sync
date:       2017-02-15
summary:    Or how we love to complicate things
categories: xamarin dotnet sqlite-net async
---
Many developers are happy using [TPL](https://www.codeproject.com/Articles/152765/Task-Parallel-Library-of-n) (Task Parallel Library) with [SQLite.Net](https://github.com/praeclarum/sqlite-net). It´s not hard to find blog posts, StackOverflow answers and other sites recommending it. The library offers an async connection that can be handy to avoid blocking the UI thread and that´s how I implemented SQLite.Net from the day one.

The reason to use [SQLiteAsyncConnection](https://github.com/praeclarum/sqlite-net/blob/master/src/SQLiteAsync.cs#L45) is pretty simple: I want my app to stay responsive when querying the database. Big queries take time. If the connection is synchronous my app may become unresponsive for a while until the data is retrieved. We care about users and C# async/await is relatively easy, so let´s use it everywhere!, right? 

## Well... sqlite is NOT async.
That´s right. Sqlite access data synchronously, no matter what. Async connections are fake :shit:. Simple as that. Taking a look at the [source code](https://github.com/praeclarum/sqlite-net/blob/master/src/SQLiteAsync.cs#L169), you´ll find out that every method call is wrapped within a Task:

{% highlight csharp %}
public Task<T> GetAsync<T>(object pk) where T : new()
{
    return Task.Factory.StartNew(() =>
    {
        var conn = GetConnection();
        using (conn.Lock())
        {
            return conn.Get<T>(pk);
        }
    });
}
{% endhighlight %}

Ok... so _what´s wrong with it?_

## You are forced to use async wrappers everywhere
You can´t choose to make something async only when it´s needed. Once you setup an async connection, you will be forced to use these wrappers everywhere unless you create a new sync connection. What if you find out that a query is fast enough to be sync?

1) Would you stick to async/await anyway?  
2) Would you create a new sync connection for those cases?  

If you choose 1, _you are complicating things_. I mean: you are awaiting a `Task` rather than a direct and simple method call, forcing your method to be async. I found out that in mobile development (mostly in UI initialization logic), async is [harder to manage](http://stackoverflow.com/search?q=xamarin+async+initialization). Database queries are usually made on screen initialization, so there you have it. You still can make it worst, by getting the result of a task in a sync fashion: `var user = connection.GetAsync<User>(1).Result` so you end up using a sync method, wrapped in a Task to be async, and then converted back to sync. Well played! :punch:

If you choose 2, _you are complicating things_. The best way to work with SQLite in most cases is reusing a single connection during the whole life cycle.

## You need to be careful with concurrency and locking errors
Just take a look [here](https://forums.xamarin.com/discussion/549/sqlite-net-and-multiple-threads) or [here](https://www.google.es/search?q=sqlite-net-pcl+sqlite+busy&oq=sqlite-net-pcl+sqlite+busy&aqs=chrome..69i57.6194j0j9&sourceid=chrome&ie=UTF-8#q=xamarin+sqlite+busy) or [here](https://bitbucket.org/twincoders/sqlite-net-extensions/issues/60/async-db-operations-sqliteexception-busy). The same can happen with sync connections, but it´s easier to fix:

{% highlight csharp %}
Connection = new SQLiteConnection(path, SQLiteOpenFlags.ReadWrite 
    | SQLiteOpenFlags.Create | SQLiteOpenFlags.FullMutex);
{% endhighlight %}

I´ve never had a single issue when creating the connection that way.

## How many of your queries really take so long to be async?
I´m pretty sure that if you [measure the time](https://github.com/Fody/MethodTimer) it takes for the most of your queries to run, you´ll realize they are surprisingly fast. Fast enough so that the user won´t notice a delay. Go ahead and try it.

## What happens with blocking, time consuming queries?
In those cases you can just make the async wrapper yourself. Your code will provide more information: from the point of view of another developer working with your code, it will be easier to know which queries are fast (sync) and which are not (async). 

## Conclusion
 <blockquote>
  <p>
    KISS is an acronym for "Keep it simple, stupid". The KISS principle states that most systems work best if they are kept simple rather than made complicated; therefore simplicity should be a key goal in design and unnecessary complexity should be avoided.
  </p>
  <footer><cite title="Wikipedia">Wikipedia</cite></footer>
</blockquote>
