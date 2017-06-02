FROM php:7.1-alpine
MAINTAINER Bart Reunes <metalarend@gmail.com>

# Install Composer globally - https://github.com/composer/composer

ENV COMPOSER_HOME /composer
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV PATH /composer/vendor/bin:$PATH

RUN curl https://getcomposer.org/installer -o /tmp/composer-setup.php \
    && curl https://composer.github.io/installer.sig -o /tmp/composer-setup.sig \
    && php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') !== trim(file_get_contents('/tmp/composer-setup.sig'))) { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('/tmp/composer-setup.php');" \
    && php -r "unlink('/tmp/composer-setup.sig');"

# Install Tini - https://github.com/krallin/tini

RUN apk add --update --no-cache tini \
    && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

# Install Xdebug - https://github.com/xdebug/xdebug

ENV XDEBUG_VERSION 2.5.3
RUN apk --update add --no-cache --virtual .build-dependencies gzip autoconf gcc g++ make \
    && cd /tmp \
    && curl -O --verbose https://xdebug.org/files/xdebug-$XDEBUG_VERSION.tgz \
    && ls -hal \
    && tar -xzf xdebug-$XDEBUG_VERSION.tgz \
    && cd xdebug-$XDEBUG_VERSION \
    && phpize && ./configure --enable-xdebug && make && make install \
    && apk del .build-dependencies \
    && rm -rf /tmp/* \
    && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > $PHP_INI_DIR/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on" >> $PHP_INI_DIR/conf.d/xdebug.ini \
    && echo "xdebug.remote_handler=dbgp" >> $PHP_INI_DIR/conf.d/xdebug.ini \
    && echo "xdebug.remote_connect_back=1" >> $PHP_INI_DIR/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=on" >> $PHP_INI_DIR/conf.d/xdebug.ini \
    && echo "xdebug.remote_port=9000" >> $PHP_INI_DIR/conf.d/xdebug.ini

# Install PHPUnit - https://github.com/sebastianbergmann/phpunit

RUN echo "memory_limit=-1" > $PHP_INI_DIR/conf.d/memory-limit.ini
RUN composer global require --dev phpunit/phpunit

# Package container

VOLUME ["/app", "/composer"]
WORKDIR "/app"
ENTRYPOINT ["/sbin/tini", "--", "phpunit"]
