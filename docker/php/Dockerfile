FROM php:8.1-fpm


# Set working directory
WORKDIR /var/www

USER root

RUN rm -rf /var/lib/apt/lists/*
RUN rm -rf /var/cache/apt/*

# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/* && mkdir -p /var/lib/apt/lists/partial

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    gnupg2 \
    curl \
    unixodbc-dev
    
# Microsoft SQL Server Prerequisites
RUN apt-get update \
    && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/11/prod.list \
        > /etc/apt/sources.list.d/mssql-release.list

# install SQL Server drivers
RUN apt-get update && ACCEPT_EULA=Y apt-get install -y msodbcsql17

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install pdo pdo_mysql exif pcntl \
    && pecl install sqlsrv pdo_sqlsrv \
    && docker-php-ext-enable sqlsrv pdo_sqlsrv
    # install SQL Server drivers

RUN apt-get update

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN getent group www || groupadd -g 1000 www
RUN getent passwd www || useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www
COPY --chown=www:www-data . /var/www/storage/logs

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
