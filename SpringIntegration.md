# Introduction #

Creating a `MemcachedClient` in a [Spring](http://www.springsource.org/) ApplicationContext used to be somewhat hairy since the `ConnectionFactoryBuilder` isn't a JavaBean. This page describes how to create a `MemcachedClient` bean using the new `MemcachedClientFactoryBean`, (since version 2.6).


# Usage #

The `net.spy.memcached.spring.MemcachedClientFactoryBean` creates a new instance of `net.spy.memcached.MemcachedClient` every time it is used.

Here's a snippet of a typical bean definition:

```
  <bean id="memcachedClient" class="net.spy.memcached.spring.MemcachedClientFactoryBean">
    <property name="servers" value="host1:11211,host2:11211,host3:11211"/>
    <property name="protocol" value="BINARY"/>
    <property name="transcoder">
      <bean class="net.spy.memcached.transcoders.SerializingTranscoder">
        <property name="compressionThreshold" value="1024"/>
      </bean>
    </property>
    <property name="opTimeout" value="1000"/>
    <property name="timeoutExceptionThreshold" value="1998"/>
    <property name="hashAlg" value="KETAMA_HASH"/>
    <property name="locatorType" value="CONSISTENT"/> 
    <property name="failureMode" value="Redistribute"/>
    <property name="useNagleAlgorithm" value="false"/>
  </bean>
```

The `MemcachedClientFactoryBean` supports the same set of attributes as the `net.spy.memcached.ConnectionFactoryBuilder` and supplies the exact same semantics:

## Servers ##
A string containing whitespace or comma separated host or IP addresses and port numbers of the form "host:port host2:port" or "host:port, host2:port".

## Daemon ##
Set the daemon state of the IO thread (defaults to true).

## FailureMode ##
Set the failure mode {Cancel | Redistribute | Retry} (defaults to Redistribute).

## HashAlg ##
Set the hash algorithm (see `net.spy.memcached.HashAlgorithm` for the values).

## InitialObservers ##
Set the initial connection observers (will observe initial connection).

## LocatorType ##
Set the locator type {ARRAY\_MOD | CONSISTENT} (defaults to ARRAY\_MOD).

## MaxReconnectDelay ##
Set the maximum reconnect delay.

## OpFact ##
Set the operation factory.

## OpQueueFactory ##
Set the operation queue factory.

## OpTimeout ##
Set the default operation timeout in milliseconds.

## Protocol ##
Convenience method to specify the protocol to use {BINARY | TEXT} (defaults to TEXT).

## ReadBufferSize ##
Set the read buffer size.

## ReadOpQueueFactory ##
Set the read queue factory.

## ShouldOptimize ##
Set to false if the default operation optimization is not desirable (defaults to true).

## Transcoder ##
Set the default transcoder (defaults to `net.spy.memcached.transcoders.SerializingTranscoder`).

## UseNagleAlgorithm ##
Set to true if you'd like to enable the Nagle algorithm.

## WriteOpQueueFactory ##
Set the write queue factory.

## AuthDescriptor ##
Set the auth descriptor to enable authentication on new connections.