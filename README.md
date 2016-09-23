# Supported tags and respective `Dockerfile` links

-   [`3.6.5`, `3.6`, `3`, `latest` (*Dockerfile*)](http://github.com/luiscoms/openshift-rabbitmq/blob/master/Dockerfile)
-   [`3.6.5-management`, `3.6-management`, `3-management`, `management` (*management/Dockerfile*)](http://github.com/luiscoms/openshift-rabbitmq/blob/master/management/Dockerfile)


This image is based on oficial [RabbitMQ image](https://hub.docker.com/_/rabbitmq/), but the base image is `openshift/base-centos7`.

Pull request to this image can be sended to the `luiscoms/openshift-rabbitmq` [GitHub repo](https://github.com/luiscoms/openshift-rabbitmq).

# What is RabbitMQ?

RabbitMQ is open source message broker software (sometimes called message-oriented middleware) that implements the Advanced Message Queuing Protocol (AMQP). The RabbitMQ server is written in the Erlang programming language and is built on the Open Telecom Platform framework for clustering and failover. Client libraries to interface with the broker are available for all major programming languages.

> [wikipedia.org/wiki/RabbitMQ](https://en.wikipedia.org/wiki/RabbitMQ)

![logo](https://raw.githubusercontent.com/docker-library/docs/master/rabbitmq/logo.png)

# How to use this image

## Running the daemon

One of the important things to note about RabbitMQ is that it stores data based on what it calls the "Node Name", which defaults to the hostname. What this means for usage in Docker is that we should specify `-h`/`--hostname` explicitly for each daemon so that we don't get a random hostname and can keep track of our data:

```console
$ docker run -d --hostname my-rabbit --name some-rabbit luiscoms/openshift-rabbitmq:3
```

If you give that a minute, then do `docker logs some-rabbit`, you'll see in the output a block similar to:

    =INFO REPORT==== 6-Jul-2015::20:47:02 ===
    node           : rabbit@my-rabbit
    home dir       : /var/lib/rabbitmq
    config file(s) : /etc/rabbitmq/rabbitmq.config
    cookie hash    : UoNOcDhfxW9uoZ92wh6BjA==
    log            : tty
    sasl log       : tty
    database dir   : /var/lib/rabbitmq/mnesia/rabbit@my-rabbit

Note the `database dir` there, especially that it has my "Node Name" appended to the end for the file storage. This image makes all of `/var/lib/rabbitmq` a volume by default.

### Erlang Cookie

See the [RabbitMQ "Clustering Guide"](https://www.rabbitmq.com/clustering.html#erlang-cookie) for more information about cookies and why they're necessary.

For setting a consistent cookie (especially useful for clustering but also for remote/cross-container administration via `rabbitmqctl`), use `RABBITMQ_ERLANG_COOKIE`:

```console
$ docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_ERLANG_COOKIE='secret cookie here' luiscoms/openshift-rabbitmq:3
```

This can then be used from a separate instance to connect:

```console
$ docker run -it --rm --link some-rabbit:my-rabbit -e RABBITMQ_ERLANG_COOKIE='secret cookie here' luiscoms/openshift-rabbitmq:3 bash
$ rabbitmqctl -n rabbit@my-rabbit list_users
Listing users ...
guest   [administrator]
```

Alternatively, one can also use `RABBITMQ_NODENAME` to make repeated `rabbitmqctl` invocations simpler:

```console
$ docker run -it --rm --link some-rabbit:my-rabbit -e RABBITMQ_ERLANG_COOKIE='secret cookie here' -e RABBITMQ_NODENAME=rabbit@my-rabbit luiscoms/openshift-rabbitmq:3 bash
$ rabbitmqctl list_users
Listing users ...
guest   [administrator]
```

### Management Plugin

There is a second set of tags provided with the [management plugin](https://www.rabbitmq.com/management.html) installed and enabled by default, which is available on the standard management port of 15672, with the default username and password of `guest` / `guest`:

```console
$ docker run -d --hostname my-rabbit --name some-rabbit luiscoms/openshift-rabbitmq:3-management
```

You can access it by visiting `http://container-ip:15672` in a browser or, if you need access outside the host, on port 8080:

```console
$ docker run -d --hostname my-rabbit --name some-rabbit -p 8080:15672 luiscoms/openshift-rabbitmq:3-management
```

You can then go to `http://localhost:8080` or `http://host-ip:8080` in a browser.

## Setting default user and password

If you wish to change the default username and password of `guest` / `guest`, you can do so with the `RABBITMQ_DEFAULT_USER` and `RABBITMQ_DEFAULT_PASS` environmental variables:

```console
$ docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password luiscoms/openshift-rabbitmq:3-management
```

You can then go to `http://localhost:8080` or `http://host-ip:8080` in a browser and use `user`/`password` to gain access to the management console

## Setting default vhost

If you wish to change the default vhost, you can do so with the `RABBITMQ_DEFAULT_VHOST` environmental variables:

```console
$ docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost luiscoms/openshift-rabbitmq:3-management
```

## Enabling HiPE

See the [RabbitMQ "Configuration"](http://www.rabbitmq.com/configure.html#config-items) for more information about various configuration options.

For enabling the HiPE compiler on startup use `RABBITMQ_HIPE_COMPILE` set to `1`. Accroding to the official documentation:

> Set to true to precompile parts of RabbitMQ with HiPE, a just-in-time compiler for Erlang. This will increase server throughput at the cost of increased startup time. You might see 20-50% better performance at the cost of a few minutes delay at startup.

It is therefore important to take that startup delay into consideration when configuring health checks, automated clustering etc.

## Use it on Openshift

```console
$ oc new-app luiscoms/openshift-rabbitmq:management
```

## Connecting to the daemon

```console
$ docker run --name some-app --link some-rabbit:rabbit -d application-that-uses-rabbitmq
```

# License

View [license information](https://www.rabbitmq.com/mpl.html) for the software contained in this image.

# Supported Docker versions

This image is officially supported on Docker version 1.12.1.

Support for older versions (down to 1.6) is provided on a best-effort basis.

Please see [the Docker installation documentation](https://docs.docker.com/installation/) for details on how to upgrade your Docker daemon.

# User Feedback

## Documentation

Documentation for this image is stored in the [`luiscoms/openshift-rabbitmq` GitHub project](https://github.com/luiscoms/openshift-rabbitmq).
You cen read the [repository's `README.md` file](https://github.com/luiscoms/openshift-rabbitmq/blob/master/README.md) before attempting a pull request.

## Issues

If you have any problems with or questions about this image, please contact us through a [GitHub issue](https://github.com/luiscoms/openshift-rabbitmq/issues).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/luiscoms/openshift-rabbitmq/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.