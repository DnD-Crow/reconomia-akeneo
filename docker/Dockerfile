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
    vim \
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
    pcntl \
    calendar \
    mbstring

RUN pecl install apcu imagick \
    && docker-php-ext-enable apcu imagick

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Copy the Apache binary to /usr/local/sbin
RUN cp /usr/sbin/apache2 /usr/local/sbin/
RUN mkdir -p /var/run/apache2 \
    && chown -R www-data:www-data /var/run/apache2
 \
    # Install Node.js and npm
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g yarn

# Set the working directory to /var/www/html
WORKDIR /home/site/wwwroot

# Clone Akeneo repository
COPY . /home/site/wwwroot
RUN chown -R www-data:www-data /home/site/wwwroot

# Set up Apache
RUN a2enmod rewrite proxy proxy_fcgi expires

COPY docker/apache/envvars /etc/apache2/envvars
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

