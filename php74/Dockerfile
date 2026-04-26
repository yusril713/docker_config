FROM php:7.4-fpm

ARG PUID=1000
ARG PGID=1000

RUN echo 'Acquire::Retries "3";' > /etc/apt/apt.conf.d/80-retries

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpng-dev \
    libjpeg-dev \
    libwebp-dev \
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
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
    && docker-php-ext-install -j$(nproc) \
        gd \
        pdo \
        pdo_mysql \
        mysqli \
        mbstring \
        xml \
        bcmath \
        intl \
        opcache \
        pcntl \
        posix \
        soap \
        zip \
        curl \
        exif \
        sockets \
        calendar \
        gettext

RUN pecl install redis apcu \
    && docker-php-ext-enable redis apcu

# Install Node.js 20 LTS
RUN curl -fsSL https://nodejs.org/dist/v20.19.1/node-v20.19.1-linux-x64.tar.gz \
    | tar -xz -C /usr/local --strip-components=1

# Install Composer 2.x
RUN curl -sS https://getcomposer.org/installer | php -- \
    --install-dir=/usr/local/bin \
    --filename=composer

COPY php.ini /usr/local/etc/php/conf.d/custom.ini

RUN groupmod -o -g ${PGID} www-data && usermod -o -u ${PUID} -g www-data www-data \
    && mkdir -p /var/www/.npm && chown -R ${PUID}:${PGID} /var/www/.npm

WORKDIR /var/www

USER www-data

EXPOSE 9000
CMD ["php-fpm"]
