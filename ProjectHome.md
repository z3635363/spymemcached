A simple, asynchronous, single-threaded memcached client written in java.

  * Efficient storage of objects.  General serializable objects are stored in their serialized form and optionally compressed if they meet criteria. Certain native objects are stored as tightly as possible (for example, a Date object generally consumes six bytes, and a Long can be anywhere from zero to eight bytes).
  * Resilient to server and network outages. In many cases, a client operation can be replayed against a server if it goes away and comes back. In cases where it can't, it will communicate that as well. An exponential backoff reconnect algorithm is applied when a memcached becomes unavailable, but asynchronous operations will queue up for the server to be applied when it comes back online.
  * Operations are asynchronous. It is possible to issue a store and continue processing without having to wait for that operation to finish. It is even possible to issue a get, do some further processing, check the result of the get and cancel it if it doesn't return fast enough.
  * There is only one thread for all processing. Regardless of the number of requests, threads using the client, or servers to which the client is connected, only one thread will ever be allocated to a given MemcachedClient.
  * Aggressively optimized. There are [many optimizations](Optimizations.md) that combine to provide high throughput.

**Note - as of the 2.9 (and 2.10) series, artifacts are published to Maven Central and the groupId has changed from "spy" to "net.spy":**

**Grab the new artifacts from here: http://search.maven.org/#browse%7C-1323196529**

**Please use at least 2.10.2 from the 2.10 series since it fixes some issues for the new 2.10.0 and 2.10.1 features!**

Version 2.10.3 was released on 2 Dev 2013
Version 2.10.2 was released on 5 Nov 2013
Version 2.10.1 was released on 11 Oct 2013
Version 2.10.0 was released on 5 Sept 2013
Version 2.9.1 was released on 4 July 2013
Version 2.9.0 was released on 4 June 2013

Please note that google code doesn't allow file uploads starting 2014. You can download the jar files directly from maven central, even if you are not using a package manager (see http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22net.spy%22%20AND%20a%3A%22spymemcached%22 under "Download").



---


YourKit is kindly supporting the spymemcached open source project with its full-featured Java Profiler.
YourKit, LLC is the creator of innovative and intelligent tools for profiling
Java and .NET applications. Take a look at YourKit's leading software products:
[YourKit Java Profiler](http://www.yourkit.com/java/profiler/index.jsp) and
[YourKit .NET Profiler](http://www.yourkit.com/.net/profiler/index.jsp)