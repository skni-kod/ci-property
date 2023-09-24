FROM nexus.skni.edu.pl:1181/php:8.2-apache

RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
  --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y \
    libpq-dev \
    libonig-dev \
    libzip-dev \
    git \
    zip \
    unzip

RUN echo "ServerName steel-dart.skni.edu.pl" >> /etc/apache2/apache2.conf

ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# 3. mod_rewrite for URL rewrite and mod_headers for .htaccess extra headers like Access-Control-Allow-Origin-
RUN a2enmod rewrite headers

RUN docker-php-ext-install pdo_pgsql mbstring exif pcntl bcmath zip

COPY --from=nexus.skni.edu.pl:1181/composer:latest /usr/bin/composer /usr/bin/composer

COPY . /var/www/html

ENV APP_ENV production
RUN composer install

RUN chown -R www-data:www-data /var/www/html/ && \
    chown -R www-data:www-data /var/www/html/bootstrap/cache && \
    chmod -R 755 /var/www/html/storage && \
    chmod -R 755 /var/www/html/bootstrap/cache

CMD php artisan config:clear && \
    php artisan config:cache && \
    php artisan optimize && \
    apache2-foreground
