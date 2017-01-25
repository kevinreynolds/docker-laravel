FROM ubuntu:16.04

MAINTAINER Alexander Diachenko

RUN locale-gen en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \

    && apt-get install -y curl cron software-properties-common \

    && add-apt-repository -y ppa:ondrej/php \
    && apt-get update \
    && apt-get install -y php7.1-cli \
        # php7.1-dev \
        php7.1-mbstring php7.1-json php7.1-curl php7.1-intl php7.1-zip php7.1-xml \
        php7.1-mysql \
        php7.1-redis \
        # php7.1-pgsql \
        # php7.1-sqlite3 \
        # php7.1-mongodb \
        # php7.1-memcached \
        # php7.1-bcmath \
        # php7.1-gd \
        # php7.1-apcu \
        # php7.1-imap \
        # php7.1-xmlrpc \
        # php7.1-soap \
        # php7.1-readline \

    && apt-get remove -y --purge software-properties-common \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PHP_DISPLAY_ERRORS ${PHP_DISPLAY_ERRORS}
ENV PHP_DISPLAY_STARTUP_ERRORS ${PHP_DISPLAY_STARTUP_ERRORS}
ENV PHP_ERROR_REPORTING ${PHP_ERROR_REPORTING}
ENV PHP_CLI_MEMORY_LIMIT ${PHP_ERROR_REPORTING}

COPY 00-cli.ini /etc/php/7.1/cli/conf.d/00-cli.ini

COPY crontabs /var/spool/cron/crontabs
RUN chmod 0600 /var/spool/cron/crontabs/root

CMD ["cron", "-f"]
