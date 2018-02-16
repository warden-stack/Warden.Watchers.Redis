# Warden Redis Watcher

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Redis.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Watchers.Redis)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Redis.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Watchers.Redis/branches)


**RedisWatcher** can be used either for simple database monitoring (e.g. checking if a connection can be made) or more advanced one which may include running a specialized query.

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Watchers.Redis)**. 
```
dotnet add package Warden.Watchers.Redis
```

### Configuration:

 - **WithQuery()** - executes the specified query on a selected database.
 - **WithTimeout()** - timeout after which the invalid result will be returned.
 - **EnsureThat()** - predicate containing a received query result of type *IEnumerable<dynamic>* that has to be met in order to return a valid result.
 - **EnsureThatAsync()** - async  - predicate containing a received query result of type *IEnumerable<dynamic>* that has to be met in order to return a valid result
 - **WithConnectionProvider()** - provide a  custom *IRedisConnection* which is responsible for making a connection to the database. 
 - **WithRedisProvider()** - provide a  custom *IRedis* which is responsible for executing a query on a database. 

Please note that either *WithConnectionProvider()* or *WithDatabaseProvider()* methods can be used to achieve the same goal, which is getting the *IRedis* instance. 

Example of configuring the watcher via provided configuration class:
```csharp
var configuration = RedisWatcherConfiguration
    .Create(1, "localhost")
    .WithQuery("get test")
    .EnsureThat(results => results.Any(x => x == "test-value"))
    .Build();
var redisWatcher = RedisWatcher.Create("Redis watcher", configuration);

var wardenConfiguration = WardenConfiguration
    .Create()
    .AddWatcher(redisWatcher )
    //Configure other watchers, hooks etc.
```

Example of adding the watcher directly to the **Warden** via one of the extension methods:
```csharp
var wardenConfiguration  = WardenConfiguration
    .Create()
    .AddRedisWatcher("localhost", 1, cfg =>
    {
        cfg.WithQuery("get test")
           .EnsureThat(results => results.Any(x => x == "test-value"));
    })
    //Configure other watchers, hooks etc.
```

Please note that you may either use the lambda expression for configuring the watcher or pass the configuration instance directly. You may also configure the **hooks** by using another lambda expression available in the extension methods.

### Check result type:
**RedisWatcher** provides a custom **RedisWatcherCheckResult** type which contains additional values.

```csharp
public class RedisWatcherCheckResult: WatcherCheckResult
{
    public int Database { get; }
    public string ConnectionString { get; }
    public string Query { get; }
    public IEnumerable<dynamic> QueryResult { get; }
}
```

### Custom interfaces:
```csharp
public interface IRedisConnection
{
    string ConnectionString { get; }
    TimeSpan Timeout { get; }
    Task<IRedis> GetDatabaseAsync(int database);
}
```

**IRedisConnection** is responsible for making a connection to the database. It can be configured via the *WithConnectionProvider()* method. By default it is based on the **[StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis)**.


```csharp
public interface IRedis
{
    Task<IEnumerable<dynamic>> QueryAsync(string query);
}
```

**IRedis** is responsible for executing the query on a database. It can be configured via the *WithRedisProvider()* method. By default, it is based on the **[StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis)**.