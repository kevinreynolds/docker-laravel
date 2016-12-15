## Installation

1. Put/clone you Laravel application in `./app` folder. 

2. Create your environment configuration: `cp .env.example .env`

3. Run `docker-compose up -d`.

Note, that `./app` folder is added to `.gitignore` which you may want to change if by any chance you choose to store docker setup and your source code in the same repo. 

It's generally recommended to contain source code in a sub directory so that you can easily add other members in the stack (i.e. single-page application or socket.io companion) later.

## Xdebug

Use [Xdebug Helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc) browser extension and PhpStorm to work with Xdebug on Docker.

You must first set `DOCKERHOST` in `.env` file so that you can connect to host machine from inside the  docker container. For example, on Windows 10 the correct value would be `10.0.75.1`.

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
                max-size: ${LOG_SIZE}
                max-file: ${LOG_ROTATE}
        volumes_from:
            - app
```

It's not included by by default for performance reasons.

Don't look for `cron` logs. There ain't any and `docker-compose logs cron` won't work. There are various hacks to circumvent this limitation when running cron daemon in Docker container and you look them up on Stackoverflow, but they are ugly (and more importantly, they make maintaining your Docker deployment much harder) so I won't be introducing them by default for now.

Note, that cron image includes PHP CLI so you might want to sync it with some of the changes in php image. You probably won't be doing this much often as a lot of php settings are configured via environment variables.

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
        environment:
            POSTGRES_DB: ${RDMS_DATABASE}
            POSTGRES_USER: ${RDMS_USER}
            POSTGRES_PASSWORD: ${RDMS_PASSWORD}
        image: postgres
        logging:
            driver: json-file
            options:
                max-size: ${LOG_SIZE}
                max-file: ${LOG_ROTATE}
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
            - ./napp:/var/www/html

    node:
        build: ./docker/build/node
        command: npm run start
        environment:
            - NODE_ENV=${NODE_ENV}
            - PORT=${NODE_PORT}
        expose:
            - ${NODE_PORT}
        links:
            - napp
            - redis
        logging:
            driver: json-file
            options:
                max-size: ${LOG_SIZE}
                max-file: ${LOG_ROTATE}
        ports:
            - ${NODE_PORT}:${NODE_PORT}
        volumes_from:
            - napp
```

You might want to change/remove default command or disable service dependency on redis (enabled by default so you don't forget it for Pub/Sub stuff).
