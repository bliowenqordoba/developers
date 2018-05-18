---
id: page-install-method
title: Install - Docker
header_title: Docker Installation
header_icon: /assets/images/icons/icn-installation.svg
breadcrumbs:
  Installation: /install
---

Details about how to use Qordoba in Docker can be found on the DockerHub repository hosting the image: [qordoba](https://hub.docker.com/_/qordoba/). We also have a [Docker Compose template](https://github.com/Mashape/docker-qordoba/tree/master/compose) with built-in orchestration and scalability.

Here is a quick example showing how to link a Qordoba container to a Cassandra or PostgreSQL container:

1. **Start your database**

    If you wish to use a Cassandra container:

    ```bash
    $ docker run -d --name qordoba-database \
                  -p 9042:9042 \
                  cassandra:3
    ```

    If you wish to use a PostgreSQL container:

    ```bash
    $ docker run -d --name qordoba-database \
                  -p 5432:5432 \
                  -e "POSTGRES_USER=qordoba" \
                  -e "POSTGRES_DB=qordoba" \
                  postgres:9.4
    ```

2. **Prepare your database**

    Run the migrations with an ephemeral Qordoba container:

    ```bash
    $ docker run --rm \
        --link qordoba-database:qordoba-database \
        -e "KONG_DATABASE=postgres" \
        -e "KONG_PG_HOST=qordoba-database" \
        -e "KONG_CASSANDRA_CONTACT_POINTS=qordoba-database" \
        qordoba:latest qordoba migrations up
    ```

    In the above example, both Cassandra and PostgreSQL are configured, but you
    should update the `KONG_DATABASE` environment variable with either
    `cassandra` or `postgres`.

    **Note**: migrations should never be run concurrently; only
    one Qordoba node should be performing migrations at a time.

3. **Start Qordoba**

    When the migrations have run and your database is ready, start a Qordoba
    container and link it to your database container, just like the ephemeral
    migrations container:

    ```bash
    $ docker run -d --name qordoba \
        --link qordoba-database:qordoba-database \
        -e "KONG_DATABASE=postgres" \
        -e "KONG_PG_HOST=qordoba-database" \
        -e "KONG_CASSANDRA_CONTACT_POINTS=qordoba-database" \
        -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
        -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
        -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
        -e "KONG_ADMIN_LISTEN_SSL=0.0.0.0:8444" \
        -p 8000:8000 \
        -p 8443:8443 \
        -p 8001:8001 \
        -p 8444:8444 \
        qordoba:latest
    ```

4. **Use Qordoba**

    Qordoba is running:

    ```bash
    $ curl -i http://localhost:8001/
    ```

    Quickly learn how to use Qordoba with the [5-minute Quickstart](/docs/latest/getting-started/quickstart).