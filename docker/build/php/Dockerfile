FROM ubuntu:16.04

MAINTAINER Alexander Diachenko

RUN locale-gen en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \

    && apt-get install -y wget curl zip unzip software-properties-common \
    && apt-get install -y sqlite3 libsqlite3-dev \

    && add-apt-repository -y ppa:git-core/ppa \
    && add-apt-repository -y ppa:ondrej/php \

    && apt-get update \

    && apt-get install -y git \

    && apt-get install -y php7.1-fpm php7.1-cli \
        # php7.1-dev \
        php7.1-mbstring \
        php7.1-json \
        php7.1-curl \
        php7.1-intl \
        php7.1-zip \
        php7.1-xml \
        php7.1-readline \

        php7.1-sqlite3 \
        php7.1-mysql \
        php7.1-redis \
        # php7.1-pgsql \
        # php7.1-mongodb \
        # php7.1-memcached \

        # php7.1-bcmath \
        # php7.1-gd \
        # php7.1-apcu \
        # php7.1-imap \
        # php7.1-xmlrpc \
        # php7.1-soap \

        php7.1-xdebug \
    && phpdismod -s cli xdebug \

    && mkdir /run/php \

    && apt-get remove -y --purge software-properties-common

RUN php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer

RUN wget https://phar.phpunit.de/phpunit.phar \
    && chmod +x phpunit.phar \
    && mv phpunit.phar /usr/local/bin/phpunit

RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - \
    && apt-get install -y nodejs

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb http://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install yarn

RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN echo 'cd /opt/project' >> ~/.bashrc

ENV TERM xterm
ENV NODE_ENV ${NODE_ENV}

ENV PHP_DISPLAY_ERRORS ${PHP_DISPLAY_ERRORS}
ENV PHP_DISPLAY_STARTUP_ERRORS ${PHP_DISPLAY_STARTUP_ERRORS}
ENV PHP_ERROR_REPORTING ${PHP_ERROR_REPORTING}
ENV PHP_REALPATH_CACHE_SIZE ${PHP_REALPATH_CACHE_SIZE}
ENV PHP_REALPATH_CACHE_TTL ${PHP_REALPATH_CACHE_TTL}

ENV PHP_CLI_MEMORY_LIMIT ${PHP_ERROR_REPORTING}

ENV PHP_FPM_MEMORY_LIMIT ${PHP_FPM_MEMORY_LIMIT}
ENV PHP_FPM_POST_MAX_SIZE ${PHP_FPM_POST_MAX_SIZE}
ENV PHP_FPM_UPLOAD_MAX_SIZE ${PHP_FPM_UPLOAD_MAX_SIZE}
ENV PHP_FPM_OPCACHE_MEMORY_CONSUMPTION ${PHP_FPM_OPCACHE_MEMORY_CONSUMPTION}
ENV PHP_FPM_OPCACHE_INTERNED_STRINGS_BUFFER ${PHP_FPM_OPCACHE_INTERNED_STRINGS_BUFFER}
ENV PHP_FPM_OPCACHE_MAX_ACCELERATED_FILES ${PHP_FPM_OPCACHE_MAX_ACCELERATED_FILES}
ENV PHP_FPM_OPCACHE_VALIDATE_TIMESTAMPS ${PHP_FPM_OPCACHE_VALIDATE_TIMESTAMPS}
ENV PHP_FPM_XDEBUG_EXTENSION ${PHP_FPM_XDEBUG_EXTENSION}

COPY .sqliterc ~/.sqliterc
COPY php-fpm.conf /etc/php/7.1/fpm/php-fpm.conf
COPY www.conf /etc/php/7.1/fpm/pool.d/www.conf
COPY 00-cli.ini /etc/php/7.1/cli/conf.d/00-cli.ini
COPY 00-fpm.ini /etc/php/7.1/fpm/conf.d/00-fpm.ini
COPY 10-opcache.ini /etc/php/7.1/fpm/conf.d/10-opcache.ini
COPY 20-xdebug.ini /etc/php/7.1/fpm/conf.d/20-xdebug.ini

EXPOSE 9000
CMD ["php-fpm7.1"]
