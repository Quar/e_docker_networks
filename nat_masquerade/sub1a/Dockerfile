FROM dockage/alpine-openrc:3.7

USER root

WORKDIR /etc/local.d

COPY ./setup_gateway.start /etc/local.d/
WORKDIR /etc/conf.d

COPY ./openrc.conf /etc/conf.d/local
RUN chmod +x /etc/local.d/setup_gateway.start && \
    rc-update add local default

WORKDIR /
