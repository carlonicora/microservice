FROM php:8.2-fpm-alpine as base

RUN set -xe  \
    && apk add --update linux-headers --no-cache --virtual .build-deps $PHPIZE_DEPS bash git supervisor curl g++ make autoconf

RUN docker-php-ext-install mysqli bcmath opcache \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && export COMPOSER_PROCESS_TIMEOUT=600 \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && addgroup -g 1000 -S dockeruser \
    && adduser -u 1000 -S dockeruser -G dockeruser

COPY ./shell/ /tmp/scripts/
RUN chmod -R 777 /tmp/scripts/

RUN touch /var/log/supervisord.log \
    && chown -R dockeruser:dockeruser /var/log/supervisord.log

RUN chown -R dockeruser:dockeruser /var/log/supervisord.log

COPY ./php-fpm/configurations/*.ini /usr/local/etc/php/conf.d/

FROM base as prd
COPY ./php-fpm/production-configurations/* /usr/local/etc/php/conf.d/
COPY ./shell/configurations/supervisord.php.conf /etc/supervisord.conf
RUN rm -rf /tmp/*
CMD ["/usr/bin/supervisord","--configuration=/etc/supervisord.conf"]

FROM base as dev
ARG XDEBUG_PORT
ARG XDEBUG_HOST
COPY ./php-fpm/development-configurations/* /usr/local/etc/php/conf.d/
COPY ./shell/configurations/supervisord.php.dev.conf /etc/supervisord.conf
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN /tmp/scripts/distributeConfig.sh /usr/local/etc/php/conf.d/zz-xdebug.ini "{XDEBUG_PORT}" "${XDEBUG_PORT}" \
    && /tmp/scripts/distributeConfig.sh /usr/local/etc/php/conf.d/zz-xdebug.ini "{XDEBUG_HOST}" "${XDEBUG_HOST}" \
RUN docker-php-source delete \
RUN rm -rf /tmp/*

CMD ["/usr/bin/supervisord","--configuration=/etc/supervisord.conf"]