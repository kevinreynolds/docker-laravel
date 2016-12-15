## Installation

1. Put/clone you Laravel application in `./app` folder. 

2. Create your environment configuration: `cp .env.example .env`

3. Run `docker-compose up -d`.

Note, that `./app` folder is added to `.gitignore` by default which you may want to change if you choose to store docker config and application in the same repo. 

It's generally recommended to contain source code in a sub directory so that you can easily add other members in the stack (i.e. single-page application or socket.io companion app) later.

## Docker Cloud

Below are some of the steps you'll need to perform to prepare your build for deployment on Docker Cloud:

1. Make each service top level member in your stack file, remove other properties such as `version`, `volumes`, `networks` and the like.
2. Remove `logging` section from service's definition.
3. Build all of your images beforehand. Deploy your source code as image instead of using data only container with mounted directory. See [Automated builds](https://docs.docker.com/docker-cloud/builds/automated-build/) secton in the Docker docs.
4. Replace all named volumes with data only containers.
5. Set `deployment_strategy: every_node` on proxy service if you want to scale beyond one host.
6. Use Cloudflare for SSL certificates. There is also a option to use automated [Letsencrypt companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion) for your nginx proxy, but the repo has been dead for a few months and it kinda overcomplicates things unnecessarily when you consider development environment.

## Other concerns

Consider disabling Transparent Huge Pages in your Host machine kernel to avoid latency and memory usage issues with Redis.

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

## Single-page applications and Node.js

> This functionality is not yet ready for production usage

As experiment, I provide a naive implementation for running node and single page applications inside Docker container. Add these services to your `docker-compose.yml` file and place your code inside `./napp` folder (it's added to .gitignore).

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

You might want to change/remove default command (`npm run start`) or disable service dependency on redis (enabled for Pub/Sub functionality).
