---
title: "Using MockServer in CircleCI builds"
---

# Using MockServer in CircleCI builds

I have written some curl commands and wanted to verify that they are working as expected.

I am using MockServer to have a flexible server where I can define the expected requests and verify that the curl commands are working correctly.

## MockServer as docker image.

The MockServer project produces [docker images](https://www.mock-server.com/where/docker.html) available on docker-hub.

## The CircleCI docker executor

It is possible to execute builds inside docker containers, using the [Docker Executor Type](https://circleci.com/docs/2.0/executor-types/#using-docker).

CircleCI provides also a way to define [multiple images](https://circleci.com/docs/2.0/executor-types/#using-multiple-docker-images): the steps will be executed in the first container in the list (primary container) and all the tools and the exposed port provided by the other containers are available on `localhost` in the primary container.

The setup is clear: 

* The primary container must be started from an image that contains `curl`. The [CircleCI base image](https://circleci.com/docs/2.0/circleci-images/#circleci-base-image) `cimg/base:2020.01` can be used.
* The MockServer image will be a secondary container.

```yaml
version: 2
jobs:
  build:
    docker:
      # Primary container: the steps will be executed in this container
      - image: cimg/base:2020.01 
      # MockServer as secondary container
      - image: mockserver/mockserver:mockserver-5.9.0

    steps:
      - run:
          name: Curl test
          # TODO: do something real instead of this placeholder command
          command: |
            curl --request PUT "http://localhost:1080/mockserver/reset"
```

## Connection refused issue

There is an issue with this setup. The "Curl test" step is failing:

```
#!/bin/bash -eo pipefail
curl --request PUT "http://localhost:1080/mockserver/reset"

curl: (7) Failed to connect to localhost port 1080: Connection refused

Exited with code exit status 7

CircleCI received exit code 7
```

![Error in CircleCI](/images/2020-03-04-circleci-failed-step.png)


Some clue can be found in the "Container mockserver/mockserver:mockserver-5.9.0" log section:

```
java  -Dfile.encoding=UTF-8 -jar /opt/mockserver/mockserver-netty-jar-with-dependencies.jar  -serverPort 1080 -logLevel INFO &



Build was canceled
```

Even if MockServer is starting quickly thanks to the usage of Netty, the CircleCI executor does not wait enough.

## Waiting until MockServer is ready

A trivial fix is to add a sleep step:

```yaml
    steps:
      - run:
          name: Wait 30 seconds
          command: sleep 30

      - run:
          name: Curl test
          # TODO: do something real instead of this placeholder command
          command: |
            curl --request PUT "http://localhost:1080/mockserver/reset"
```

A better solution is to use `curl` with the `--retry` flag in order to wait until the MockServer reset call can be performed.
Because at the beginning the connection is refused, we need to add `--retry-connrefused` to tell `curl` to continue to try to execute the call even if the first errors are "Connection refused" ones.

The first step becomes:

```yaml
    steps:
      - run:
          name: Wait for MockServer
          command: |
            curl --retry 20 --retry-connrefused --request PUT "http://localhost:1080/mockserver/reset"

      - run:
          name: Curl test
          # TODO: do something real instead of this placeholder command
          command: |
            curl --request PUT "http://localhost:1080/mockserver/reset"
```

The complete example will be published soon. I will update this page as soon as the repository is online.
