FROM php:8.4-fpm

RUN apt-get update && apt-get install -y --no-install-recommends \
    supervisor \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libicu-dev \
    libxml2-dev \
    libzip-dev \
    libonig-dev \
    libcurl4-openssl-dev \
    zip \
    unzip \
    git \
    curl \
    && docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_mysql \
        mbstring \
        bcmath \
        pcntl \
        posix \
        zip \
        curl \
        sockets \
    && pecl install redis apcu \
    && docker-php-ext-enable redis apcu \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/log/supervisor /etc/supervisor/conf.d

COPY supervisord.conf /etc/supervisor/supervisord.conf

WORKDIR /var/www

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
