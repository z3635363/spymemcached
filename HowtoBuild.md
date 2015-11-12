# HOWTO Build #

After you GitTheCode, you'll probably want to build it.  This guide exists to give you the shortest and easiest path to do so.

## Prerequisites ##

I'm using [Apache buildr](http://buildr.apache.org) as my build systems.

There are various reasons, but it's the best thing I can find.  Ant requires too much customization, and nobody's been able to actually make my (relatively simple) build work in maven 2, so here it lies.

The least painful way to get it seems to be to use [jruby](http://jruby.org/).

Once you have jruby installed, you should have a `jgem` command, into which you will enter the following command:

```
jgem install buildr
```

You will also need a recent (1.4.1 or greater) version of [memcached](http://code.google.com/p/memcached/) running on `localhost` port `11211`.

## Doing the Build ##

Should just be as simple as

```
buildr
```