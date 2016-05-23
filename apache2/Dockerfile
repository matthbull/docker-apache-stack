FROM ubuntu:latest

MAINTAINER Matt Holbrook-Bull <matt@ampisoft.com>

RUN apt-get update

# Install apache, PHP, and supplimentary programs. curl and lynx-cur are for debugging the container.
RUN apt-get -y --allow-change-held-packages install git wget apache2 libapache2-mod-php7.0 php7.0-mysql php7.0-gd php-pear php7.0-curl php7.0-common php7.0- curl lynx-cur php7.0-json php7.0-sqlite3 php7.0-xml php7.0-common php7.0-cli php7.0-cgi php7.0 php7.0-pgsql php7.0-ldap php7.0-dev

# Enable apache mods.
RUN a2enmod php7.0
RUN a2enmod rewrite
RUN a2enmod ssl

#### install redis and xdebug extensions for php7
# redis
RUN git clone https://github.com/phpredis/phpredis.git \
    && cd phpredis \
    && git checkout php7 \
    && phpize \
    && ./configure \
    && make && make install \
    && cd .. \
    && rm -rf phpredis

RUN echo "extension=redis.so" > /etc/php/7.0/mods-available/redis.ini \
    && ln -sf /etc/php/7.0/mods-available/redis.ini /etc/php/7.0/apache2/conf.d/20-redis.ini \
    && ln -sf /etc/php/7.0/mods-available/redis.ini /etc/php/7.0/cli/conf.d/20-redis.ini 

# xdebug (not on cli to avoid composer issues
RUN wget -c "http://xdebug.org/files/xdebug-2.4.0.tgz" \
    && tar -xf xdebug-2.4.0.tgz \
    && cd xdebug-2.4.0 \
    && phpize \
    && ./configure \
    && make && make install

RUN echo "zend_extension=xdebug.so" > /etc/php/7.0/mods-available/xdebug.ini \
    && ln -sf /etc/php/7.0/mods-available/xdebug.ini /etc/php/7.0/apache2/conf.d/20-xdebug.ini

# Update the PHP.ini file
ADD php.ini /etc/php/7.0/apache2/php.ini

# Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('SHA384', 'composer-setup.php') === '92102166af5abdb03f49ce52a40591073a7b859a86e8ff13338cf7db58a19f7844fbc0bb79b2773bf30791e935dbd938') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');"

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

EXPOSE 80 443 9009 

# Copy site into place.
ADD ssl/server.crt /etc/apache2/ssl/server.crt
ADD ssl/server.key /etc/apache2/ssl/server.key
ADD www /var/www/vhosts

# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/sbs.dev.conf

# By default, simply start apache.
CMD /usr/sbin/apache2ctl -D FOREGROUND

