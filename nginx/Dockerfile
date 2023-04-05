ARG NGINX_VERSION
ARG timezone

FROM nginx:${NGINX_VERSION}

ENV TIMEZONE=${timezone:-"Asia/Shanghai"}

COPY nginx.conf /etc/nginx/

RUN ln -fs /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo "${TIMEZONE}" > /etc/timezone
RUN apk update \
    && apk upgrade \
    && apk --update add logrotate \
    && apk add --no-cache openssl \
    && apk add --no-cache bash

RUN apk add --no-cache curl

RUN set -x ; \
    addgroup -g 82 -S www-data ; \
    adduser -u 82 -D -S -G www-data www-data && exit 0 ; exit 1