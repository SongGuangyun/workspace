ARG PHP_VERSION
ARG IMAGE_TAG=latest
FROM laradock/php-fpm:${IMAGE_TAG}-${PHP_VERSION}

ARG PHP_VERSION

RUN sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    apt-get install -yqq \
    apt-utils \
    gnupg2 \
    git \
    #
    #--------------------------------------------------------------------------
    # Mandatory Software's Installation
    #--------------------------------------------------------------------------
    #
    # Mandatory Software's such as ("mcrypt", "pdo_mysql", "libssl-dev", ....)
    # are installed on the base image 'laradock/php-fpm' image. If you want
    # to add more Software's or remove existing one, you need to edit the
    # base image (https://github.com/Laradock/php-fpm).
    #
    # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
    libzip-dev zip unzip && \
    if [ ${PHP_VERSION} = "7.3" ] || [ ${PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    docker-php-ext-configure zip; \
    else \
    docker-php-ext-configure zip --with-libzip; \
    fi && \
    # Install the zip extension
    docker-php-ext-install zip && \
    php -m | grep -q 'zip'


###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    pecl install -o -f redis-4.3.0; \
    else \
    pecl install -o -f redis; \
    fi \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis \
    ;fi

###########################################################################
# Swoole EXTENSION
###########################################################################

ARG INSTALL_SWOOLE=false
RUN set -eux; \
    if [ ${INSTALL_SWOOLE} = true ]; then \
    # Install Php Swoole Extension
    if   [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "50600" ]; then \
    pecl install swoole-2.0.10; \
    elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70000" ]; then \
    pecl install swoole-4.3.5; \
    elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70100" ]; then \
    pecl install swoole-4.5.11; \
    elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
    pecl install swoole-4.8.12; \
    else \
    pecl install swoole; \
    fi; \
    docker-php-ext-enable swoole; \
    php -m | grep -q 'swoole'; \
    fi


###########################################################################
# AMQP:
###########################################################################

ARG INSTALL_AMQP=false

RUN set -eux; \
    if [ ${INSTALL_AMQP} = true ]; then \
    # # Install the amqp extension
    apt-get -yqq install librabbitmq-dev; \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    pecl install amqp-1.11.0beta; \
    else \
    pecl install amqp; \
    fi; \
    docker-php-ext-enable amqp; \
    php -m | grep -oiE '^amqp$'; \
    fi



###########################################################################
# pcntl
###########################################################################

ARG INSTALL_PCNTL=false
RUN if [ ${INSTALL_PCNTL} = true ]; then \
    # Installs pcntl, helpful for running Horizon
    docker-php-ext-install pcntl \
    ;fi

###########################################################################
# bcmath:
###########################################################################

ARG INSTALL_BCMATH=false

RUN if [ ${INSTALL_BCMATH} = true ]; then \
    # Install the bcmath extension
    docker-php-ext-install bcmath \
    ;fi




###########################################################################
# Opcache:
###########################################################################

ARG INSTALL_OPCACHE=false

RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
    ;fi

# Copy opcache configration
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

###########################################################################
# Mysqli Modifications:
###########################################################################

ARG INSTALL_MYSQLI=false

RUN if [ ${INSTALL_MYSQLI} = true ]; then \
    docker-php-ext-install mysqli \
    ;fi


###########################################################################
# Human Language and Character Encoding Support:
###########################################################################

ARG INSTALL_INTL=false

RUN if [ ${INSTALL_INTL} = true ]; then \
    # Install intl and requirements
    apt-get install -yqq zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl \
    ;fi



###########################################################################
# Image optimizers:
###########################################################################

USER root

ARG INSTALL_IMAGE_OPTIMIZERS=false

RUN if [ ${INSTALL_IMAGE_OPTIMIZERS} = true ]; then \
    apt-get install -yqq jpegoptim optipng pngquant gifsicle \
    ;fi


###########################################################################
# ImageMagick:
###########################################################################

USER root

ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -yqq libmagickwand-dev imagemagick && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    cd /tmp && \
    if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
    git clone https://github.com/Imagick/imagick; \
    else \
    git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
    fi && \
    cd imagick && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    rm -r /tmp/imagick; \
    else \
    pecl install imagick; \
    fi && \
    docker-php-ext-enable imagick; \
    php -m | grep -q 'imagick' \
    ;fi



###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

# Configure locale.
ARG LOCALE=POSIX
ENV LC_ALL ${LOCALE}

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
