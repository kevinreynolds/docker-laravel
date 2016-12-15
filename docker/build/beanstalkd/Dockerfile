FROM ubuntu:16.04

MAINTAINER Alexander Diachenko

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y beanstalkd \

    && apt-get install -y wget \
    && wget https://github.com/src-d/beanstool/releases/download/v0.2.0/beanstool_v0.2.0_linux_amd64.tar.gz \
    && tar -xvzf beanstool_v0.2.0_linux_amd64.tar.gz \
    && cp beanstool_v0.2.0_linux_amd64/beanstool /usr/local/bin/ \
    && rm -rf beanstool_v0.2.0_linux_amd64 \
    && rm beanstool_v0.2.0_linux_amd64.tar.gz \
    && apt-get remove -y --purge wget \

    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

EXPOSE 11300
CMD ["beanstalkd"]
