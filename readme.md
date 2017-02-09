## Installation

1. Put/clone you Laravel application in `./app` folder. 

2. Create your environment configuration: `cp .env.example .env`

3. Create your docker environment configuration: `cp docker.env.example docker.env`

4. Run `docker-compose up -d` to start your docker containers.

Note, that `./app` folder is added to `.gitignore` which you may want to change if by any chance you choose to store docker setup and your source code in the same repo. 

## Alternative installation

It's generally recommended to contain source code in a sub directory so that you can easily add other members in the stack later (i.e. node application or SPA). If you have a more standard Laravel setup though, you can just follow the guide bellow.

1. Copy `docker/`, `docker-compose.yml` and `docker.env.example` to the root of your Laravel repository.
2. Run `cp docker.env.example docker.env` in your project root
2. Edit `.gitignore` to include `docker.env` file
3. Edit your `.env` and `.env.example` files to include values from the `.env.example` file in this repository
4. Replace `./app:/opt/project` with `./:/opt/project` in your docker-compose.yml
5. Run `docker-compose up -d` to start your docker containers.

## Virtual hosts

Remove this section from nginx service definition in `docker-compose.yml` or replace `PROXY_PORT` with other variable value to free port 80:

```
        ports:
            - ${PROXY_PORT}:80
```

Add this proxy service:

```
    proxy:
        env_file: docker.env
        image: jwilder/nginx-proxy
        logging:
            driver: json-file
            options:
                max-size: ${SERVICE_LOG_SIZE}
                max-file: ${SERVICE_LOG_ROTATE}
        ports:
            - ${PROXY_PORT}:80
        volumes:
            - /var/run/docker.sock:/tmp/docker.sock:ro
```

Add this to your `docker.env` file. You can replace the values below with your hostnames of choice.

```
VIRTUAL_HOST=localhost,.localtunnel.me,.ngrok.io,.xip.io
```

> If you're working on Windows machine, it's possible you will encounter an [issue related to exposed 3306 port](https://github.com/jwilder/nginx-proxy/issues/699) when jwilder/nginx-proxy and mysql services are run simultaneously. This issue requires further investigation, but for now you can monkey patch it by exposing MySQL on other port inside the network. [See exponse property reference](https://docs.docker.com/compose/compose-file/#expose). Or alternatively, you separate these 2 services by network (not tested). The bug was first spotted after upgrading Docker to 1.13.0.

## Mailhog

Add Mailhog service (the application itself should connect on port `1025`):

```
    mailhog:
        image: mailhog/mailhog:latest
        logging:
            driver: json-file
            options:
                max-size: ${SERVICE_LOG_SIZE}
                max-file: ${SERVICE_LOG_ROTATE}
        ports:
            - ${MAILHOG_PORT}:8025
```

edit `.env` file:

```
MAILHOG_PORT=8025
```

## Redis

Add redis service

```
    redis:
        build: ./docker/build/redis
        command: redis-server /usr/local/etc/redis/redis.conf ${REDIS_PARAMETERS}
        logging:
            driver: json-file
            options:
                max-size: ${SERVICE_LOG_SIZE}
                max-file: ${SERVICE_LOG_ROTATE}
        ports:
            - ${REDIS_PORT}:6379
        volumes:
            - redis:/data
```

and corresponding volume

```
    redis:
        driver: local
```

Add the variables below to your `.env` file.

```
REDIS_PORT=6379

# Example: "--requirepass secret --maxmemory 100mb --maxmemory-policy allkeys-lru"
REDIS_PARAMETERS=
```

## Xdebug

Use [Xdebug Helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc) browser extension and PhpStorm to work with Xdebug on Docker.

You must first set `XDEBUG_HOST` in `.env` file so that you can connect to host machine from inside the  docker container. For example, on Windows 10 the correct value would be `10.0.75.1`.

You may temporarily mount this volume on php service to get an easy access to Xdebug Profiler Snapshots if you need one

```
        volumes:
            - ./docker/run/php/tmp:/tmp
```

## Cron

Add cron service to `docker-compose.yml`:

```
    cron:
        build: ./docker/build/cron
        links:
            - app
        logging:
            driver: json-file
            options:
                max-size: ${SERVICE_LOG_SIZE}
                max-file: ${SERVICE_LOG_ROTATE}
        volumes_from:
            - app
```

It's not included by by default for performance reasons.

Don't look for `cron` logs. There ain't any and `docker-compose logs cron` won't work. There are various hacks to circumvent this limitation when running cron daemon in Docker container and you can look them up on Stackoverflow, but they are ugly (and more importantly, they make maintaining your Docker deployment much harder) so I won't be introducing them by default for now.

**Note, that cron image includes PHP CLI so you might want to sync it with some of the changes in the main php image.** You probably won't be doing this much often as a lot of php settings are configured via environment variables.

## Beanstalkd

Add beanstalkd to services section. Note that you might want to mount `/var/lib/beanstalkd/data` as named volume if you wan to persist beanstalkd queues.

```
    beanstalkd:
        build: ./docker/build/beanstalkd
        links:
            - php
```

Image is build with [Beanstool](https://github.com/src-d/beanstool) already installed (i.e. running `docker-compose exec beanstalkd beanstool stats` will display queue stats).

## SQLite

SQLite is installed by default on php image.

## Postgres

If you wish to use postgres, simply replace mysql's service and volume with the following snippet. You must also *build PHP with pgsql extension* (see corresponding Dockerfile).

**Service**

```
    postgres:
        env_file: .env
        image: postgres
        logging:
            driver: json-file
            options:
                max-size: ${SERVICE_LOG_SIZE}
                max-file: ${SERVICE_LOG_ROTATE}
        ports:
            - "{POSTGRES_PORT}:5432"
        volumes:
            - postgres:/var/lib/postgresql/data
```

**Volume**

```
    postgres:
        driver: local
```

Add this to `.env` file:

```
POSTGRES_PORT=5432
```

Add this to your `docker.env` file:

```
POSTGRES_DB=homestead
POSTGRES_USER=homestead
POSTGRES_PASSWORD=secret
```

## MariaDB

If you wish to use mariadb instead of mysql, simply replace mysql image with mariadb's (i.e. `mariadb:10`). You don't need to change anything else.

## Single-page applications and Node.js

> This functionality is not yet ready for production usage

I provide a very naive implementation for running node and single page applications inside Docker container. Add these services to your `docker-compose.yml` file and place your code inside `./napp` folder (it's already added to .gitignore).

```
    napp:
        image: alpine
        command: /bin/sh
        volumes:
            - ./napp:/opt/project

    node:
        build: ./docker/build/node
        command: npm run start
        env_file: docker.env
        expose:
            - ${NODE_PORT}
        links:
            - napp
            - redis
        logging:
            driver: json-file
            options:
                max-size: ${SERVICE_LOG_SIZE}
                max-file: ${SERVICE_LOG_ROTATE}
        ports:
            - ${NODE_PORT}:${NODE_PORT}
        volumes_from:
            - napp
```

Add this line to `docker.env` file:

```
NODE_PORT=3000
```

You might want to change/remove default command or disable service dependency on redis (enabled by default so you don't forget it for Pub/Sub stuff).
