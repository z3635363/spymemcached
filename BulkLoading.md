# Introduction #

Since spymemcached performs all of it's mutation operations asynchronously, when bulk loading you can inadvertently overload the JVM with allocated objects which need to be sent.  The default behavior of the client is to immediately timeout operations when the bounded input queue is overrun, which can mean portions of your bulk load will need to retry.

However, with a bit of tuning, you can change this behavior and have a simple, high throughput bulk load.

TODO: add details on the bulk loader in there.

# Details #

To change the behavior of the input queue, you need simply to change the timeout value on the Queue being used by the connection.  This is done through the `ConnectionFactoryBuilder` class.  This is discussed for the Couchbase Client (which builds on this client) on the [wiki](http://www.couchbase.com/wiki/display/couchbase/Couchbase+Java+Client+Library#CouchbaseJavaClientLibrary-BulkLoading) for that client.