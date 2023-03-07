ARG NGINX_VERSION
ARG timezone

FROM nginx:${NGINX_VERSION}

ENV TIMEZONE=${timezone:-"Asia/Shanghai"}

COPY nginx.conf /etc/nginx/

RUN ls -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    # && sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list \
    # && apt-get update \
    # && apt-get install -y vim wget