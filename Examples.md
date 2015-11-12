# Simple Example #

```
MemcachedClient c=new MemcachedClient(
    new InetSocketAddress("hostname", portNum));

// Store a value (async) for one hour
c.set("someKey", 3600, someObject);
// Retrieve a value (synchronously).
Object myObject=c.get("someKey");
```

# Taking Advantage of Asynchronous Gets #

MemcachedClient may be processing a great deal of asynchronous messages or possibly dealing with an unreachable memcached, which may delay processing. If a memcached is disabled, for example, MemcachedConnection will continue to attempt to reconnect and replay pending operations until it comes back up. To prevent this from causing your application to hang, you can use one of the asynchronous mechanisms to time out a request and cancel the operation to the server.

```
// Get a memcached client connected to several servers
MemcachedClient c=new MemcachedClient(
	AddrUtil.getAddresses("server1:11211 server2:11211"));

// Try to get a value, for up to 5 seconds, and cancel if it doesn't return
Object myObj=null;
Future<Object> f=c.asyncGet("someKey");
try {
    myObj=f.get(5, TimeUnit.SECONDS);
} catch(TimeoutException e) {
    // Since we don't need this, go ahead and cancel the operation.  This
    // is not strictly necessary, but it'll save some work on the server.
    f.cancel(false);
    // Do other timeout related stuff
}
```

# Establishing Connections #

## Establishing a Binary Protocol Connection ##

As of release 2.0, the new binary wire protocol for memcached is supported. Below is an example of how you might connect to a server using the binary protocol:

```
// Get a memcached client connected to several servers with the binary protocol
MemcachedClient c=new MemcachedClient(
	new BinaryConnectionFactory(),
	AddrUtil.getAddresses("server1:11212 server2:11212"));

// Proceed to do cool stuff with memcached using the binary protocol.
```

## Establishing a Binary Protocol SASL Connection ##
```
// Create an AuthDescriptor, this one is for plain SASL, so the 
//   username and passwords are just Strings
AuthDescriptor ad = new AuthDescriptor(new String[]{"PLAIN"},
				new PlainCallbackHandler(username, password));

// Then connect using the ConnectionFactoryBuilder.  Binary is required.
        try {
            if (mc == null) {
                mc = new MemcachedClient(
                        new ConnectionFactoryBuilder().setProtocol(Protocol.BINARY)
                        .setAuthDescriptor(ad)
                        .build(),
                        AddrUtil.getAddresses(host));
            }
        } catch (IOException ex) {
            System.err.println("Couldn't create a connection, bailing out: \nIOException " + ex.getMessage());
        }
```

# Using CAS #

The low-level CAS operation is a bit difficult to get right, so I've provided an abstraction that will allow one to more easily use it.

Here's an example of how you might write a function that maintains a list of the 10 most recent `Item`s using CAS (where `Item` is some kind of object that is meaningful within your application).  Every call to `addAnItem` will place the new `Item` at the beginning of a `List` and store it (using CAS) in memcached.

The important part to focus on here is the interaction between the "mutation" (a functional construct that describes how to transform an existing value into the desired one) and the client.

Essentially, you only have to think of two things:

  1. What do I do if there's no value under the key (initialize).
  1. How do I want to manipulate an existing value if I find one?

Note that the mutation may occur multiple times (up to 8,192 by default) when there's contention.  In general, you shouldn't care about the retries and just think about what you want to do when you find a value you like.

This function returns the newly manipulated and stored list.

```
public List<Item> addAnItem(final Item newItem) throws Exception {

    // This is how we modify a list when we find one in the cache.
    CASMutation<List<Item>> mutation = new CASMutation<List<Item>>() {

        // This is only invoked when a value actually exists.
        public List<Item> getNewValue(List<Item> current) {
            // Not strictly necessary if you specify the storage as
            // LinkedList (our initial value isn't), but I like to keep
            // things functional anyway, so I'm going to copy this list
            // first.
            LinkedList<Item> ll = new LinkedList<Item>(current);

            // If the list is already "full", pop one off the end.
            if(ll.size() > 10) {
                ll.removeLast();
            }
            // Add mine first.
            ll.addFirst(newItem);

            return ll;
        }

    };

    // The initial value -- only used when there's no list stored under
    // the key.
    List<Item> initialValue=Collections.singletonList(newItem);

    // The mutator who'll do all the low-level stuff.
    CASMutator<List<Item>> mutator = new CASMutator<List<Item>>(client, transcoder);

    // This returns whatever value was successfully stored within the
    // cache -- either the initial list as above, or a mutated existing
    // one
    return mutator.cas("myKey", initialValue, 0, mutation);
}
```