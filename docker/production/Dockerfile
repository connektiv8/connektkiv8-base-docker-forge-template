FROM node:18 as build_frontend

WORKDIR /app
COPY . /app
RUN npm install
RUN npm run build
RUN rm -rf node_modules

FROM composer:2 as build_composer
WORKDIR /app
COPY --from=build_frontend /app /app
RUN composer install --no-dev --ignore-platform-reqs --no-interaction

FROM ubuntu:22.04

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y nano cron nginx \
       gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
       && apt-get install -y php8.2-cli php8.2-dev \
       php8.2-pgsql php8.2-sqlite3 php8.2-gd php8.2-imagick \
       php8.2-curl \
       php8.2-imap php8.2-mysql php8.2-mbstring \
       php8.2-xml php8.2-zip php8.2-bcmath php8.2-soap \
       php8.2-intl php8.2-readline \
       php8.2-ldap \
       php8.2-msgpack php8.2-igbinary php8.2-redis php8.2-swoole \
       php8.2-memcached php8.2-pcov php8.2-xdebug \
    && apt-get update \
    && apt-get install -y mysql-client \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# USER
RUN groupadd --force -g 1000 forge
RUN useradd -ms /bin/bash --no-user-group -g 1000 -u 1000 forge
RUN usermod -aG sudo forge
RUN usermod -aG forge www-data

# NGINX
COPY ./docker/production/nginx.conf /etc/nginx/nginx.conf
COPY ./docker/production/nginx_default.conf /etc/nginx/sites-available/default

# PHP
COPY ./docker/production/php-fpm.conf /etc/php/8.2/fpm/php-fpm.conf
COPY ./docker/production/www.conf /etc/php/8.2/fpm/pool.d/www.conf
COPY ./docker/production/php.ini /etc/php/8.2/cli/conf.d/99-php.ini

# SERVICES
COPY ./docker/production/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# PROJECT FILES OWNERSHIP AND PERMISSIONS
RUN chown -R forge:forge /var/www
COPY --from=build_composer --chown=forge:forge /app /var/www/html

EXPOSE 80

CMD ["/usr/bin/supervisord"]
