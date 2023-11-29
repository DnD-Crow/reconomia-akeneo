# Use an official PHP image with Apache and PHP 8.1
FROM php:8.1-fpm-bullseye

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libbz2-dev \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libicu-dev \
    libonig-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libwebp-dev \
    libpq-dev \
    libmagickwand-dev \
    libsodium-dev \
    apache2 \
    aspell \
    ghostscript \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

# Configure GD extension
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp

# Install PHP extensions
RUN docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    intl \
    zip \
    opcache \
    bcmath \
    sodium \
    exif \
    gd \
    mbstring

RUN pecl install apcu imagick \
    && docker-php-ext-enable apcu imagick

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Copy the Apache binary to /usr/local/sbin
RUN cp /usr/sbin/apache2 /usr/local/sbin/

# Install Node.js and npm
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs

# Set the working directory to /var/www/html
WORKDIR /var/www/html

# Clone Akeneo repository
COPY . /var/www/html

# Install Akeneo dependencies using Composer
# RUN composer install --prefer-dist --no-dev --no-scripts

# Install Node.js dependencies
# RUN npm install

# Build front-end assets
# RUN npm run webpack

# Set up Apache
RUN a2enmod rewrite proxy proxy_fcgi

COPY docker/apache/apache2.conf /etc/apache2/apache2.conf
COPY docker/apache/sites-enabled/akeneo.conf /etc/apache2/sites-available/000-default.conf
COPY docker/apache/conf-enabled/charset.conf /etc/apache2/conf-enabled/charset.conf
COPY docker/apache/conf-enabled/other-vhosts-access-log.conf /etc/apache2/conf-enabled/other-vhosts-access-log.conf
COPY docker/apache/conf-enabled/prod_tuning.conf /etc/apache2/conf-enabled/prod_tuning.conf
COPY docker/apache/conf-enabled/security.conf /etc/apache2/conf-enabled/security.conf
COPY docker/apache/conf-enabled/serve-cgi-bin.conf /etc/apache2/conf-enabled/serve-cgi-bin.conf

# Supervisor configuration
COPY docker/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Expose the Apache port
EXPOSE 80

# Start Supervisor to manage processes
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

