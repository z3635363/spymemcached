# Logging? #

For reasons now relegated to history, Spy has its own logging implementation.  However, it is compatible with other types of logging, if configured as such.

# Logging Howto #

spymemcached is built on top of an internal logging API that has some nice logging abstractions built in.  It's analogous to what you might find in Apache's commons-logging, except Dustin had his for quite a while before finding that one.

Both log4j and Java's built-in logging are supported.  The logger is selected via the system property `net.spy.memcached.compat.log.LoggerImpl`.

## Using log4j ##

Set the logger impl to `net.spy.log.Log4JLogger`.  For example:

```
  -Dnet.spy.log.LoggerImpl=net.spy.memcached.compat.log.Log4JLogger 
```

## Using Java's Built-in Logging ##

Set the logger impl to `net.spy.memcached.compat.log.SunLogger`.  For example:

```
  -Dnet.spy.log.LoggerImpl=net.spy.memcached.compat.log.SunLogger
```

This can also be done programmatically, as shown below.

If you're writing a simple application, say a test, and you simply want to get the default console handler to be more verbose, you can set that up like so:

```

	// Tell spy to use the SunLogger
	Properties systemProperties = System.getProperties();
	systemProperties.put("net.spy.log.LoggerImpl", "net.spy.memcached.compat.log.SunLogger");
	System.setProperties(systemProperties);

	Logger.getLogger("net.spy.memcached").setLevel(Level.FINEST);

	//get the top Logger
	Logger topLogger = java.util.logging.Logger.getLogger("");

	// Handler for console (reuse it if it already exists)
	Handler consoleHandler = null;
	//see if there is already a console handler
	for (Handler handler : topLogger.getHandlers()) {
	    if (handler instanceof ConsoleHandler) {
		//found the console handler
		consoleHandler = handler;
		break;
	    }
	}

	if (consoleHandler == null) {
	    //there was no console handler found, create a new one
	    consoleHandler = new ConsoleHandler();
	    topLogger.addHandler(consoleHandler);
	}

	//set the console handler to fine:
	consoleHandler.setLevel(java.util.logging.Level.FINEST);

```

## Making Logging More Verbose with JDK Logger ##

Sometimes, you want to log what's happening with the internals of spymemcached, but not for every class.  An easy way to do that is to define some properties and pass in more specific logging properties.  For instance, if you start the JVM with -Djava.util.logging.config.file=logging.properties defined and then put the text below in a file named logging.properties in your classpath, you can log for just the `net.spymemcached.vbucket` classes.

```
net.spy.memcached.vbucket.level = FINEST
```

---

Attribution:
The java.util.logging method of getting the consoleHandler was borrowed from [this stackoverflow thread](http://stackoverflow.com/questions/470430/java-util-logging-logger-doesnt-respect-java-util-logging-level)