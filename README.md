# WildFly over OpenJ9 Docker image

Conteiner docker com [WildFly application server](http://wildfly.org/) rodando sobre a [JVM](https://adoptopenjdk.net/) [OpenJ9](https://www.eclipse.org/openj9/index.html)

## Usage

To boot in standalone mode

    docker run -it hermanolira/wildfly-openj9
    
To boot in standalone mode with admin console available remotely

    docker run -p 8080:8080 -p 9990:9990 -it hermanolira/wildfly-openj9 /opt/jboss/wildfly/bin/standalone.sh -bmanagement 0.0.0.0

To boot in domain mode

    docker run -it hermanolira/wildfly-openj9 /opt/jboss/wildfly/bin/domain.sh -b 0.0.0.0 -bmanagement 0.0.0.0

## Application deployment

With the WildFly server you can [deploy your application in multiple ways](https://docs.jboss.org/author/display/WFLY8/Application+deployment):

1. You can use CLI
2. You can use the web console
3. You can use the management API directly
4. You can use the deployment scanner

The most popular way of deploying an application is using the deployment scanner. In WildFly this method is enabled by default and the only thing you need to do is to place your application inside of the `deployments/` directory. It can be `/opt/jboss/wildfly/standalone/deployments/` or `/opt/jboss/wildfly/domain/deployments/` depending on [which mode](https://docs.jboss.org/author/display/WFLY8/Operating+modes) you choose (standalone is default in the `jboss/wildfly` image -- see above).

The simplest and cleanest way to deploy an application to WildFly running in a container started from the `jboss/wildfly` image is to use the deployment scanner method mentioned above.

To do this you just need to extend the `hermanolira/wildfly-openj9` image by creating a new one. Place your application inside the `deployments/` directory with the `ADD` command (but make sure to include the trailing slash on the deployment folder path, [more info](https://docs.docker.com/reference/builder/#add)). You can also do the changes to the configuration (if any) as additional steps (`RUN` command).  

[A simple example](https://github.com/goldmann/wildfly-docker-deployment-example) was prepared to show how to do it, but the steps are following:

1. Create `Dockerfile` with following content:

        FROM hermanolira/wildfly-openj9
        ADD your-awesome-app.war /opt/jboss/wildfly/standalone/deployments/
2. Place your `your-awesome-app.war` file in the same directory as your `Dockerfile`.
3. Run the build with `docker build --tag=wildfly-app .`
4. Run the container with `docker run -it wildfly-app`. Application will be deployed on the container boot.

This way of deployment is great because of a few things:

1. It utilizes Docker as the build tool providing stable builds
2. Rebuilding image this way is very fast (once again: Docker)
3. You only need to do changes to the base WildFly image that are required to run your application

## Logging

Logging can be done in many ways. [This blog post](https://goldmann.pl/blog/2014/07/18/logging-with-the-wildfly-docker-image/) describes a lot of them.

## Customizing configuration

Sometimes you need to customize the application server configuration. There are many ways to do it and [this blog post](https://goldmann.pl/blog/2014/07/23/customizing-the-configuration-of-the-wildfly-docker-image/) tries to summarize it.

## Extending the image

To be able to create a management user to access the administration console create a Dockerfile with the following content

    FROM hermanolira/wildfly-openj9
    RUN /opt/jboss/wildfly/bin/add-user.sh admin Admin#70365 --silent
    CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]

Then you can build the image:

    docker build --tag=hermanolira/wildfly-openj9-admin .

Run it:

    docker run -it hermanolira/wildfly-openj9-admin

Administration console will be available on the port `9990` of the container.

## Building on your own

You don't need to do this on your own, because we prepared a trusted build for this repository, but if you really want:

    docker build --rm=true --tag=hermanolira/wildfly-openj9 .

## Image internals [updated Oct 14, 2014]

This image extends the [`adoptopenjdk/openjdk8-openj9:latest`](https://github.com/AdoptOpenJDK/openjdk-docker/blob/master/8/jdk/ubuntu/Dockerfile.openj9) image which adds the OpenJDK distribution on top of the [`ubuntu:16.04`](https://github.com/tianon/docker-brew-ubuntu-core/blob/58cc180042b7ebec2b683576faa00c04d5d011e2/xenial/Dockerfile) image. Please refer to the README.md for selected images for more info.

The server is run as the `jboss` user which has the uid/gid set to `1000`.

WildFly is installed in the `/opt/jboss/wildfly` directory.

## Source

The source is [available on GitHub](https://github.com/hermanolira/wildfly-openj9).
