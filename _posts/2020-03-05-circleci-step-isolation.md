---
title: "CircleCI step isolation"
---

# CircleCI step isolation 

To use MockServer inside a CircleCI build, using the [docker executor with MockServer as secondary container](2020-03-04-use-mock-sever-in-circleci-builds.md) works well.

In this article I explored an other way: start MockServer as java process.

From a CircleCI perspective you need to run in a docker container or on a machine that that provides a JVM.
With the docker executor, you can run using the docker image `circleci/openjdk:8-jdk` for your primary container.
The machine executor using the image `ubuntu-1604:201903-01` is also working well (but is slower).

## Mock-Server as Java process

Because MockServer is written in java, it is possible to run it using `java -jar`.
One of the [published artifact](https://www.mock-server.com/where/maven_central.html) is a convenient jar containing all the dependencies, so that you have no issue with the java classpath.
It is available using following coordinates:

```xml
<dependency>
  <groupId>org.mock-server</groupId>
  <artifactId>mockserver-netty</artifactId>
  <version>5.9.0</version>
  <classifier>jar-with-dependencies</classifier>
  <packaging>jar</packaging>
</dependency>
```

This all-in-one jar can be fetched directly from the corresponding url (for version `5.9.0`):

```
https://repo1.maven.org/maven2/org/mock-server/mockserver-netty/5.9.0/mockserver-netty-5.9.0-jar-with-dependencies.jar
```

An other idea is to use maven to perform the download, this way the artifact is also cached locally (the [maven local repository can be reused between builds](https://circleci.com/docs/2.0/caching/#maven-java-and-leiningen-clojure)):

```
mvn org.apache.maven.plugins:maven-dependency-plugin:3.1.1:copy \
    -Dartifact=org.mock-server:mockserver-netty:5.9.0:jar:jar-with-dependencies \
    -DoutputDirectory=build
```

The previous command will put the `mockserver-netty-5.9.0-jar-with-dependencies.jar` file inside a `build/` folder.
We can start it using the [java command line](https://www.mock-server.com/mock_server/running_mock_server.html#running_from_command_line_using_java):

```
java -jar build/mockserver-netty-5.9.0-jar-with-dependencies.jar -serverPort 1080
```

But if we are using maven, we can also use the [Maven Plugin form the command line](https://www.mock-server.com/mock_server/running_mock_server.html#running_from_command_line_using_maven_plugin) (without any maven build):

```
mvn -Dmockserver.serverPort=1080 -Dmockserver.logLevel=INFO org.mock-server:mockserver-maven-plugin:5.9.0:runForked
```

## The CircleCI step isolation

My first version was more or less looking like this (I removed the steps to cache and restore the local maven repository):

```yaml
version: 2
jobs:
  build:
    docker:
      - image: circleci/openjdk:8-jdk

    steps:
      - run:
          name: Start mock-server
          command: mvn -Dmockserver.serverPort=1080 -Dmockserver.logLevel=INFO org.mock-server:mockserver-maven-plugin:5.9.0:runForked

      - run:
          name: Curl test
          command: |
            curl --request PUT "http://localhost:1080/mockserver/reset"

      - run:
          name: Stop mock-server
          command: mvn -Dmockserver.serverPort=1080 org.mock-server:mockserver-maven-plugin:5.9.0:stopForked
          when: always
```

But I got the error:

```
#!/bin/bash -eo pipefail
curl --request PUT "http://localhost:1080/mockserver/reset"

curl: (7) Failed to connect to localhost port 1080: Connection refused

Exited with code exit status 7

CircleCI received exit code 7
```

![Error in CircleCI](/images/2020-03-04-circleci-failed-step.png)

And it seems that I am not the only one:

* [Can reach localhost using curl either inside or outside docker container](https://discuss.circleci.com/t/can-reach-localhost-using-curl-either-inside-or-outside-docker-container/15967)
* [Cannot curl to localhost: connection refused](https://discuss.circleci.com/t/cannot-curl-to-localhost-connection-refused/17885)
* ...

It was difficult for me to understand why this wasn't working.

Even if I did not find a precise statement in the [steps reference guide](https://circleci.com/docs/2.0/configuration-reference/#steps), my conclusion is that because each steps is using a separated shell, the operations started during one step might no longer be present in the next steps.

It works as expected when I start MockServer and perform the `curl` command inside the same step:

```yaml
version: 2
jobs:
  build:
    docker:
      - image: circleci/openjdk:8-jdk

    steps:
      - run:
          command: mvn -Dmockserver.serverPort=1080 -Dmockserver.logLevel=INFO org.mock-server:mockserver-maven-plugin:5.9.0:runForked

      - run:
          name: Single step
          command: |
            mvn -Dmockserver.serverPort=1080 -Dmockserver.logLevel=INFO org.mock-server:mockserver-maven-plugin:5.9.0:runForked
            curl --request PUT "http://localhost:1080/mockserver/reset"
            mvn -Dmockserver.serverPort=1080 org.mock-server:mockserver-maven-plugin:5.9.0:stopForked
```

There is also a `background` key on the steps that can be used at one specific step.
I did not try it, but I imagine that processes started during a background step continue to run during the next steps.
This could also be a possibility, but in this case you need to wait for MockServer to be started using `sleep` or "`curl` with retries" as explained in my article where [MockServer is started as secondary container](2020-03-04-use-mock-sever-in-circleci-builds.md).