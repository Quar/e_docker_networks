FROM dockage/alpine-openrc:3.7

USER root

RUN apk add --no-cache iptables

WORKDIR /etc/local.d
COPY ./setup_iptables.start /etc/local.d/

WORKDIR /etc/conf.d
COPY ./openrc.conf /etc/conf.d/local

RUN chmod +x /etc/local.d/setup_iptables.start && \
    rc-update add local default

WORKDIR /
