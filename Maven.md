# Using spymemcached with Maven #

## Since 2.9 ##

**NOTE: as of 2.9, you can grab the maven artifacts directly from Maven Central!**


http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22net.spy%22

## Until 2.8.12 ##

Maven artifacts are hosted for spymemcached.

## spy Maven Repository ##

The Maven2 repository for spymemcached is hosted at:
http://files.couchbase.com/maven2/

An older repository is at http://bleu.west.spy.net/~dustin/m2repo/.  It's not done yet, but the artifacts from that will be consolidated into the repository above.

## Using the Maven Repository ##

The repository is easily useable by tools that can either scan the repository or use a Nexus index.  If an index URL is requested, use "http://files.couchbase.com/maven2/.index".  Many tools will just default to this directory for you when a URL is specified.


In Apache [buildr](http://buildr.apache.org/) you can specify the repository like this:

```
repositories.remote << "http://files.couchbase.com/maven2/"
```

In maven, you specify it like this:

```
  <repositories>
    <repository>
      <id>spy</id>
      <name>Spy Repository</name>
      <layout>default</layout>
      <url>http://files.couchbase.com/maven2/</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
```

## The Artifact ##

The actual artifact is specified as follows:

```
    <dependency>
    	<groupId>spy</groupId>
    	<artifactId>spymemcached</artifactId>
    	<version>2.8.1</version>
    	<scope>provided</scope>
    </dependency>
```