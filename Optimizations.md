# Optimization Overview #

There are several elements of the design that each allow high throughput. Each is discussed below along with an example case showing many of them working together.

# Single-threaded IO #

The IO thread mirrors the server-side design of memcached by multiplexing asynchronous IO across multiple connections to multiple servers.

Each MemcachedClient instance establishes and maintains a single connection to each server in your cluster. Data are sent as soon as they become available and are able to be received by the remote sides. Responses are similarly collected as soon as they arrive.

# Very Low Contention #

> There are two points where client threads (i.e. your code) and the IO thread meet. Whenever a caller needs to issue a request against memcached, it does so by building an object that represents the request which it queues into a `java.util.concurrent.BlockingQueue` instance (with a non-blocking insertion). A `java.util.concurrent.Future` is returned to the caller.

When a request is complete and the response is available, the IO thread feeds it back into this future.

The contention on the enqueuing is as small as java's concurrency utilities allow (in practice, this is quite low). As for the result, the typical scenario involves a single thread waiting on a latch. It's hard to imagine a case where either of these would contribute a detectable amount of latency.

# Asynchronous Interface #

With an asynchronous interface, it's possible to “fire and forget” requests such as sets and deletes. You can optionally wait for the results, but if it's not necessary for your application, I won't make you do it.

For every enqueued request, there's a `java.lang.concurrent.Future` returned that is used to track the progress of the request.  All of the communication from the IO thread to the callers is done by way of futures. This also allows you to do things like processing between a get request and the response from it.

# Multi-get Escalation #

In the process of finding data to write over the wire, the IO thread will notice when there are multiple get requests in a row and collapse them into a single multi-get request with deduplicated keys.

For example, several outstanding requests in a queue for a single server that look like this: `[a], [b], [a, b, c], [a], [d]`, will be collapsed into a single request for `[a, b, c, d]` and then the results will be delivered to the respective callers (five in this case).

# Protocol Pipelining #

Another effect of having a single connection to each server is that the process of converting a request to the wire format doesn't actually care about the requests themselves, so it can effectively ignore natural boundaries.

For example, if the queue contains a few gets, sets and deletes, it's possible to send all of those in a single packet.

# Altogether Now #

Consider an example where a memcached instance has two values, `x` and `y`, both set to `1`.  No other values exist within this instance. Six threads issue requests as shown in the diagram below at approximately the same time, and get queued top to bottom.

![http://spymemcached.googlecode.com/svn/wiki/MemcachedOptimization.png](http://spymemcached.googlecode.com/svn/wiki/MemcachedOptimization.png)

  1. The first request will immediately hit the wire since enqueuing a request causes the loop to look for IO. This returns the value `1` to `t1`. `t1` is happy and goes home.
  1. The next time through the loop, more items have made it into the queue, so they're processed together (at least, until the transmit buffer is full).
    1. The next time we're ready to transmit, the IO thread will notice that there are three gets in a row (the optimizer does not look past anything that will mutate data), and combine them into a single get. The gets have overlapping keys, so the keys will be deduplicated so that the write buffer receives a multi-get for the keys `y` and `z`.
    1. The write buffer is not full, so we look to the input queue again and see there's a set ready, so we add that to the buffer.
    1. The write buffer is still not full, so we look at the input queue yet again and see another get. There are no more gets, so we enqueue this one.
    1. The write buffer is not full, but the input queue is empty, so we transmit this data.
  1. The multi-get for `y` and `z` returns, so we send `t2` and `t4` the value `1` for `y` and `t3` a `null` for `z` (since `z` was not in our cache). Threads `t2`, `t3`, and `t4` now have all of the necessary results.
  1. The set returns and the results (we'll call it a success) are sent to the future given to `t5`. If `t5` cares about the result, it now has it. If it doesn't, it's off doing something else by now.
  1. Finally, the second get request for `y` returns, this time with the value `2`, which is provided to the `t6`.

# See Also: #

