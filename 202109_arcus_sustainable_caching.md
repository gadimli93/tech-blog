# Sustainable Caching Method in ARCUS and How To Apply It

In large-scale applications for general users, when retrieving data (especially hot data), a high volume of requests towards the DB 
will result in a high load. Therefore a general method to reduce the load on the DB side and to provide a fast response is storing
the frequently retrieved data in the distributed cache of an application.

When it comes to applying cache to the application Demand-fill caching pattern is the most commonly used method.
When an application requests data, first it will be checked in the cache-store, if data exists in the cache,
retrieve data from the cache, otherwise, data will be retrieved from the database, stored into the cache, and after that,
it will be returned. Hence a fast response can only be provided if data exists in the cache. Please check the 
[ARCUS Common Cache Module Use with Basic Pattern Caching in Java Environment](https://github.com/gadimli93/tech-blog/blob/main/202011_arcus_common_module.md) 
for more details on the demand-fill method.

However, the problem with this method is that when data in the database has been modified, this update won't be reflected 
on the data in the cache. Therefore to reduce the data mismatch between DB and cache **Expire Time** or **Time-To-Live(TTL)** is set. 
Because of this characteristic following issues may occur.

1. The difference in response speed to the request, in the cases of caching and expired caching.
2. DB load with a high volume of requests, right after the cache data expire until the data is cached again.

These are the problems that occur when the cached data has expired and all requests go to the database, especially the second issue is referred to as cache stampede.

## Cache Stampede

<img src="images/202109_arcus_sustainable_caching_stampede.jpeg"></img>

A stampede is a situation in which a group of large animals suddenly start running in the same direction in a sudden panicked rush.
