FROM php:7.2-apache

# add repository for libreoffice-writer
RUN echo 'deb http://security.debian.org/debian-security jessie/updates main' >> /etc/apt/sources.list && \
    echo 'deb http://ftp.de.debian.org/debian jessie main' >> /etc/apt/sources.list && \
    echo 'deb http://ftp.de.debian.org/debian jessie-updates main' >> /etc/apt/sources.list

# needed to install tzdata below
RUN mkdir -p /usr/share/man/man1

# install basic tools
RUN apt update && \
    apt install -y --allow-downgrades wget \
    vim \
    git \
    htop \
    xpdf \
    supervisor \
    mysql-client \
    # we only need this specific version of tzdata to install openjdk-(7|8)-jre
    tzdata=2019a-0+deb8u1 \
    tzdata-java \
    openjdk-7-jre-headless \
    libreoffice-writer \
    # needed to add the sources for nodejs
    gnupg2

# install nodejs libs to run minify and uglify
RUN apt install -y apt-transport-https && \
    curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt update && \
    apt install -y nodejs yarn && \
    npm install -g uglify-js \
    uglifycss \
    mammoth

# install ruby and dependencies
RUN apt install -y ruby \
    ruby-dev && \
    gem update --system && \
    gem install json \
    json_pure \
    compass \
    sass

# external libraries for php
RUN apt install -y libxml2 \
    libxml2-dev \
    libmcrypt-dev \
    libicu-dev \
    libpng12-dev \
    libc-client-dev \
    libkrb5-dev \
    librabbitmq-dev

# configure, install, and enable php extensions which came inside original php docker image
RUN docker-php-ext-configure pdo_mysql --with-pdo-mysql=mysqlnd && \
    docker-php-ext-install \
    pdo_mysql \
    pdo && \
    docker-php-ext-configure gd && \
    docker-php-ext-install gd && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install imap \
    opcache \
    xml \
    intl \
    mbstring \
    bcmath \
    json \
    iconv \
    soap \
    zip \
    exif \
    gettext

# install and enable php redis client and rabbitmq
RUN pecl install redis-3.1.6 && \
    echo '' | pecl install amqp && \
    docker-php-ext-enable redis

# install imagick
RUN apt install -y libmagickwand-dev \
    && pecl install imagick-beta \
    && echo "extension=imagick.so" > /usr/local/etc/php/conf.d/ext-imagick.ini

# install composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

RUN usermod -s /bin/bash -u 1000 -d /var/www/app www-data && \
    chgrp -R www-data /tmp && chmod -R ug+rwx /tmp && \
    wget https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh -O /usr/local/bin/wait-for-it.sh && \
    chmod 755 /usr/local/bin/wait-for-it.sh

ADD --chown=www-data:www-data https://raw.githubusercontent.com/php/php-src/php-7.2.0/php.ini-development /usr/local/etc/php/php.ini

COPY files/app.test.conf /etc/apache2/sites-available/
RUN a2ensite app.test.conf && \
    a2enmod rewrite

# disable xdebug
RUN sed -Ei "s/^short_open_tag.+/short_open_tag = On/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^max_execution_time.+/max_execution_time = 0/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^;date.timezone.+/date.timezone = \"UTC\"/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^;sendmail_path.+/sendmail_path = \/usr\/sbin\/sendmail -t -i/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^memory_limit.+/memory_limit = 2G/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^output_buffering.+/output_buffering = On/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^post_max_size.+/post_max_size = 300M/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^upload_max_filesize.+/upload_max_filesize = 256M/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^;realpath_cache_size.+/realpath_cache_size = 2M/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^;realpath_cache_ttl.+/realpath_cache_ttl = 86400/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^;\s?max_input_vars.+/max_input_vars = 50000/" /usr/local/etc/php/php.ini && \
    sed -Ei "s/^opcache\.interned_strings_buffer.+/opcache.interned_strings_buffer = 64/" /usr/local/etc/php/php.ini && \
    echo "extension=amqp.so" > /usr/local/etc/php/conf.d/docker-php-ext-amqp.ini

RUN rm -rf /var/lib/apt/lists/*

ENV APP_ROOT="/var/www/app"

WORKDIR /var/www/app
