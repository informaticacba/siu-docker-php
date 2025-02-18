FROM alpine:3.7 AS sodium
RUN apk add \
    php7 \
    php7-pear \
    php7-dev \
    gcc \
    musl-dev \
    make \
    libsodium \
    libsodium-dev \
    && pecl install -f libsodium-1.0.6

FROM alpine:3.7 as cli

# Por defecto se utiliza la timezone de Buenos Aires
ENV TZ=America/Argentina/Buenos_Aires

RUN apk --no-cache add \
    bash \
    tzdata \
    php7 \
    php7-ctype \
    php7-curl \
    php7-dom \
    php7-exif \
    php7-fileinfo \
    php7-gd \
    php7-gmp \
    php7-iconv \
    php7-json \
    php7-mbstring \
    php7-mcrypt \
    php7-memcached \
    php7-opcache \
    php7-openssl \
    php7-pcntl \
    php7-pdo_pgsql \
    php7-phar \
    php7-session \
    php7-simplexml \
    php7-tokenizer \
    php7-zip \
    php7-xml \
    php7-xmlwriter \
    php7-xsl \
    libsodium \
    curl \
    && rm -rf /var/cache/apk/* \
    && echo "date.timezone  = $TZ" > /etc/php7/conf.d/80-siu-timezone.ini \
    && echo extension=libsodium.so > /etc/php7/conf.d/00_sodium.ini

COPY --from=sodium /usr/lib/php7/modules/libsodium.so /usr/lib/php7/modules/

FROM cli as web

RUN apk --no-cache add \
    apache2 \
    php7-apache2 \
    && rm -rf /var/cache/apk/* \
    && mkdir -p /var/log/apache2 \
    && mkdir -p /var/run/apache2 \
    && ln -s /dev/stderr /var/log/apache2/error.log \
    && ln -s /dev/stdout /var/log/apache2/access.log
COPY common/apache/* /etc/apache2/conf.d/
COPY common/php/* /etc/php7/conf.d/
COPY common/siu-entrypoint.d/* /siu-entrypoint.d/
COPY common/entrypoint.sh /usr/local/bin/entrypoint.sh

ENTRYPOINT ["entrypoint.sh"]

CMD ["httpd", "-D", "FOREGROUND"]

EXPOSE 80

FROM web as rootless

ARG USER=siu
ENV USER=$USER
ARG UID=222

RUN adduser -h /usr/local/app -u $UID -G www-data -S $USER \
    && chown -R $USER.0 \
        /var/log/apache2 \
        /var/run/apache2 \
        /etc/php7/conf.d \
        /etc/apache2/conf.d \
        /var/www/localhost/htdocs \
        /siu-entrypoint.d \
    && chmod -R g=u \
        /etc/passwd \
        /var/log/apache2 \
        /var/run/apache2 \
        /etc/php7/conf.d \
        /etc/apache2/conf.d \
        /var/www/localhost/htdocs \
        /siu-entrypoint.d
USER $USER
