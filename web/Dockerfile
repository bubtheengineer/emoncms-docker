# Offical Docker PHP & Apache image https://hub.docker.com/_/php/
FROM php:8.0-apache

# Install deps
RUN apt-get update && apt-get install -y \
              libcurl4-gnutls-dev \
              libmcrypt-dev \
              gettext \
              nano \
              git-core \
              supervisor \
              python3-pip

# Enable PHP modules
RUN docker-php-ext-install -j$(nproc) mysqli gettext

# Install python redis module
RUN pip install redis

# Install redis
COPY install_redis.sh /
RUN chmod +x /install_redis.sh && \
    /install_redis.sh

# Install mosquitto
COPY install_mosquitto.sh /
RUN chmod +x /install_mosquitto.sh && \
    /install_mosquitto.sh

RUN a2enmod rewrite

# Add custom PHP config
COPY config/php.ini /usr/local/etc/php/

# Add custom Apache config
COPY config/emoncms.conf /etc/apache2/sites-available/emoncms.conf
RUN a2dissite 000-default.conf
RUN a2ensite emoncms

# NOT USED ANYMORE - GIT CLONE INSTEAD
# Copy in emoncms files, files can be mounted from local FS for dev see docker-compose
# ADD ./emoncms /var/www/html

# Clone in master Emoncms repo & modules - overwritten in development with local FS files
RUN mkdir /var/www/emoncms && \
    git clone https://github.com/emoncms/emoncms.git /var/www/emoncms && \
    git clone https://github.com/emoncms/dashboard.git /var/www/emoncms/Modules/dashboard && \
    git clone https://github.com/emoncms/graph.git /var/www/emoncms/Modules/graph && \
    git clone https://github.com/emoncms/app.git /var/www/emoncms/Modules/app && \
    git clone https://github.com/emoncms/device.git /var/www/emoncms/Modules/device && \
    git clone https://github.com/emoncms/sync.git /home/pi/sync

RUN chgrp www-data /home/pi && \
    chgrp -R www-data /home/pi/sync && \
    ln -s /var/www/html /var/www/emoncms && \
    ln -s /home/pi/sync/sync-module /var/www/emoncms/Modules/sync

COPY docker.settings.ini /var/www/emoncms/settings.ini

# Create folders & set permissions for feed-engine data folders (mounted as docker volumes in docker-compose)
RUN mkdir /var/opt/emoncms && \
    mkdir /var/opt/emoncms/phpfina && \
    mkdir /var/opt/emoncms/phptimeseries && \
    chown www-data:root /var/opt/emoncms/phpfina && \
    chown www-data:root /var/opt/emoncms/phptimeseries

# Create Emoncms logfile
RUN mkdir /var/log/emoncms && \
    touch /var/log/emoncms/emoncms.log && \
    chmod 666 /var/log/emoncms/emoncms.log

# To start Apache and emoncms_mqtt from supervisord
COPY config/supervisord.conf /etc/supervisor/supervisord.conf
ENTRYPOINT [ "/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf" ]
