########################################################################################################################
### DEFAULT BUILD ARGUMENTS
### They can be overwritten in docker-compose.yml under services.container_name.build.args
########################################################################################################################
ARG NEST_PHP_VERSION
FROM composer:latest as composer
########################################################################################################################
FROM php:${NEST_PHP_VERSION}

ARG NEST_PHALCON_VERSION
ARG NEST_PSR_VERSION
ARG NEST_PHALCON_DEVTOOLS_VERSION
ARG INSTALL_DEVTOOLS
ARG INSTALL_NEWRELIC
ARG INSTALL_FFMPEG
ARG PHALCON_EXT_PATH=php7/64bits

LABEL maintainer="Nazarii Sliusarchuk <ns@leobit.com>"

# Install Phalcon and PSR
RUN set -xe && \
        # Download PSR, see https://github.com/jbboehr/php-psr
        curl -LO https://github.com/jbboehr/php-psr/archive/v${NEST_PSR_VERSION}.tar.gz && \
        tar xzf ${PWD}/v${NEST_PSR_VERSION}.tar.gz && \
        # Download Phalcon
        curl -LO https://github.com/phalcon/cphalcon/archive/v${NEST_PHALCON_VERSION}.tar.gz && \
        tar xzf ${PWD}/v${NEST_PHALCON_VERSION}.tar.gz && \
        docker-php-ext-install -j $(getconf _NPROCESSORS_ONLN) \
            ${PWD}/php-psr-${NEST_PSR_VERSION} \
            ${PWD}/cphalcon-${NEST_PHALCON_VERSION}/build/${PHALCON_EXT_PATH} \
        && rm -r \
            ${PWD}/v${NEST_PSR_VERSION}.tar.gz \
            ${PWD}/v${NEST_PHALCON_VERSION}.tar.gz \
            ${PWD}/php-psr-${NEST_PSR_VERSION} \
            ${PWD}/cphalcon-${NEST_PHALCON_VERSION}

########################################################################################################################
# Install Phalcon DevTools
########################################################################################################################
RUN if [ $INSTALL_DEVTOOLS = true ]; then \
    curl -LOs https://github.com/phalcon/phalcon-devtools/releases/download/v${NEST_PHALCON_DEVTOOLS_VERSION}/phalcon.phar && \
    chmod +x phalcon.phar && \
    mv phalcon.phar /usr/local/bin/phalcon \
;fi

########################################################################################################################
# Install Debian dependencies required for PHP and PECL libraries
########################################################################################################################
ARG PECL_DEPENDENCIES="libgmp-dev libmagickwand-dev libpng-dev libtidy-dev libzstd-dev libyaml-dev libzip-dev"
ARG PECL_EXT="redis igbinary apcu yaml imagick xdebug"
ARG PHP_EXT="mysqli pdo_mysql exif gettext gd tidy gmp zip"
RUN apt-get update && \
    apt-get -yq upgrade && \
    apt-get install -y --no-install-recommends apt-utils git zip unzip $PECL_DEPENDENCIES && \
    docker-php-ext-install $PHP_EXT && \
    pecl install -o -f $PECL_EXT && \
    pecl clear-cache && \
    docker-php-ext-enable $PECL_EXT && \
    apt-get remove --purge -y $PECL_DEPENDENCIES;

########################################################################################################################
# Install NewRelic daemon
########################################################################################################################
ARG PHP_AGENT_URL="https://download.newrelic.com/php_agent/release/newrelic-php5-9.18.1.303-linux.tar.gz"
RUN if [ $INSTALL_NEWRELIC = true ]; then \
    curl -L ${PHP_AGENT_URL} | tar -C /tmp -zx && \
    export NR_INSTALL_USE_CP_NOT_LN=1 && \
    export NR_INSTALL_SILENT=1 && \
    /tmp/newrelic-php5-*/newrelic-install install && \
    rm -rf /tmp/newrelic-php5-* /tmp/nrinstall* \
;fi

# Generate SSL certificates
ARG SSL_SUBJECT="/C=CA/ST=QC/O=ACME/CN=localhost"
RUN openssl req -x509 -nodes -days 365 \
    -subj "${SSL_SUBJECT}" \
    -newkey rsa:2048 -keyout /etc/ssl/private/localhost.key \
    -out /etc/ssl/certs/localhost.crt;
# Apache configuration
ENV APACHE_DOCUMENT_ROOT /var/www
ENV APACHE_PUBLIC_DOCUMENT_ROOT /var/www/html
RUN sed -ri -e 's!/var/www/html!${APACHE_PUBLIC_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
RUN a2enmod rewrite headers expires ssl

# Composer Install
COPY --from=composer /usr/bin/composer /usr/local/bin/composer

WORKDIR /home