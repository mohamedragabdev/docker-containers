FROM php:8.5

WORKDIR /app

RUN apt update && apt install -y zip unzip

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN docker-php-ext-install pdo_mysql

CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]