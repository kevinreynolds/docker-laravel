FROM node:boron

MAINTAINER Alexander Diachenko

ARG DEBIAN_FRONTEND=noninteractive

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb http://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install yarn \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV NODE_ENV ${NODE_ENV}

WORKDIR /opt/project

CMD ["bash"]
