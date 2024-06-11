FROM ubuntu:22.04

ARG OLS_VERSION
ARG PHP_VERSION
ARG TARGETPLATFORM

RUN apt-get update && apt-get install wget curl cron tzdata -y

# RUN wget https://github.com/litespeedtech/openlitespeed/releases/download/v1.8.1/openlitespeed-$OLS_VERSION-x86_64-linux.tgz && \
#     tar xzf openlitespeed-$OLS_VERSION.tgz && cd openlitespeed && ./install.sh && \
#     echo 'cloud-docker' > /usr/local/lsws/PLAT && rm -rf /openlitespeed && rm /openlitespeed-$OLS_VERSION.tgz

RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
    wget https://github.com/litespeedtech/openlitespeed/releases/download/v$OLS_VERSION/openlitespeed-$OLS_VERSION-x86_64-linux.tgz && tar xzf openlitespeed-$OLS_VERSION-x86_64-linux.tgz && cd openlitespeed && ./install.sh && cp ./admin/misc/lsup.sh /usr/local/lsws/admin/misc/lsup.sh && chmod +x /usr/local/lsws/admin/misc/lsup.sh && echo 'cloud-docker' > /usr/local/lsws/PLAT && rm -rf /openlitespeed && rm /openlitespeed-$OLS_VERSION-x86_64-linux.tgz; \
  elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
    wget https://github.com/litespeedtech/openlitespeed/releases/download/v$OLS_VERSION/openlitespeed-$OLS_VERSION-aarch64-linux.tgz && tar xzf openlitespeed-$OLS_VERSION-aarch64-linux.tgz && cd openlitespeed && ./install.sh && cp ./admin/misc/lsup.sh /usr/local/lsws/admin/misc/lsup.sh && chmod +x /usr/local/lsws/admin/misc/lsup.sh && echo 'cloud-docker' > /usr/local/lsws/PLAT && rm -rf /openlitespeed && rm /openlitespeed-$OLS_VERSION-aarch64-linux.tgz; \
  fi


# RUN apt-get install autoconf g++ ghostscript imagemagick unzip default-mysql-client linux-headers-generic libfreetype6 libicu-dev libssl-dev libjpeg-turbo8 libwebp-dev libpng-dev libzip-dev libssh2-1 git libgomp1 libssh2-1-dev lz4 libavif-dev libxpm4 lz4 $PHP_VERSION $PHP_VERSION-common $PHP_VERSION-mysql $PHP_VERSION-opcache $PHP_VERSION-curl $PHP_VERSION-imagick $PHP_VERSION-redis $PHP_VERSION-memcached $PHP_VERSION-intl -y

RUN apt-get install mysql-client $PHP_VERSION $PHP_VERSION-common $PHP_VERSION-mysql $PHP_VERSION-opcache \
    $PHP_VERSION-curl $PHP_VERSION-imagick $PHP_VERSION-redis $PHP_VERSION-memcached $PHP_VERSION-intl -y

RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == lsphp7* ]]; then apt-get install $PHP_VERSION-json -y; fi"]

# RUN wget -O /usr/local/lsws/admin/misc/lsup.sh \
#     https://raw.githubusercontent.com/litespeedtech/openlitespeed/master/dist/admin/misc/lsup.sh && \
#     chmod +x /usr/local/lsws/admin/misc/lsup.sh


RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
&& chmod +x wp-cli.phar && mv wp-cli.phar /usr/bin/wp \
&& ln -s /usr/local/lsws/$PHP_VERSION/bin/php /usr/bin/php \
&& wp package install wp-cli/doctor-command:@stable --allow-root \
&& mkdir -p /home/nobody/.wp-cli \
&& mv ~/.wp-cli/packages /home/nobody/.wp-cli/ \
&& chown -R nobody:nogroup /home/nobody/.wp-cli 

# RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && \
# chmod +x wp-cli.phar && mv wp-cli.phar /usr/bin/wp && \
# ln -s /usr/local/lsws/$PHP_VERSION/bin/php /usr/bin/php

# Install PHP extensions
RUN wget -O - https://repo.litespeed.sh | sh
RUN apt-get install $PHP_VERSION-apcu $PHP_VERSION-opcache -y

RUN wget -O -  https://get.acme.sh | sh

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 7080
ENV PATH="/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/bin"

ADD docker.conf /usr/local/lsws/conf/templates/docker.conf
ADD setup_docker.sh /usr/local/lsws/bin/setup_docker.sh
ADD httpd_config.xml /usr/local/lsws/conf/httpd_config.xml
ADD htpasswd /usr/local/lsws/admin/conf/htpasswd

RUN /usr/local/lsws/bin/setup_docker.sh && rm /usr/local/lsws/bin/setup_docker.sh
RUN chown 999:999 /usr/local/lsws/conf -R
RUN cp -RP /usr/local/lsws/conf/ /usr/local/lsws/.conf/
RUN cp -RP /usr/local/lsws/admin/conf /usr/local/lsws/admin/.conf/
#RUN sed -i "s|fcgi-bin/lsphp|/usr/local/lsws/$PHP_VERSION/bin/lsphp|g" /usr/local/lsws/conf/httpd_config.conf
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == lsphp8* ]]; then ln -sf /usr/local/lsws/$PHP_VERSION/bin/lsphp /usr/local/lsws/fcgi-bin/lsphp8; fi"]
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == lsphp8* ]]; then ln -sf /usr/local/lsws/fcgi-bin/lsphp8 /usr/local/lsws/fcgi-bin/lsphp; fi"]
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == lsphp7* ]]; then ln -sf /usr/local/lsws/$PHP_VERSION/bin/lsphp /usr/local/lsws/fcgi-bin/lsphp7; fi"]
RUN ["/bin/bash", "-c", "if [[ $PHP_VERSION == lsphp7* ]]; then ln -sf /usr/local/lsws/fcgi-bin/lsphp7 /usr/local/lsws/fcgi-bin/lsphp; fi"]

# Set pager for WP-CLI
ENV PAGER=less

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /var/www/vhosts/
