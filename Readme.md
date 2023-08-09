# apos configurar a parte do docker no terminal digitar docker compose up 
## lebrando que o docker tem que esta aberto

## Laravel - Depois de tudo configurado podemos instalar o laravel 
## docker-compose run --rm app composer create-project --prefer-dist laravel/laravel application

## Apos a instação deve se colocar no docker-compose abaixo de 
##  - ./application:/var/www/application

o comando 

 environment:
      - COMPOSER_HOME=/composer
      - COMPOSER_ALLOW_SUPERUSER=1
      - APP_ENV=local
      - APP_KEY= aqui vai a chave gerada pelo .env do laravel.

## feito isso deve reiniciar o docker 

docker compose down
docker compose up

## feito isso digite na barra de navegação http://localhost:8000 o Docker e laravel estara pronto.


## application e nome da aplição pode mudar o nome mais mais precisa ser alterado no projeto exe:

server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/application/public; -> mudar aqui 
    
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;  # Alterado para o nome do serviço PHP no Docker Compose
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}


--------------------------------------

FROM php:8.2-fpm

RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    jpegoptim optipng pngquant gifsicle \
    vim \
    git \
    curl \
    zip \
    unzip \
    libpq-dev \
    libzip-dev \
    libexif-dev \
    libonig-dev 

WORKDIR /var/www

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-install gd

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Adjust permissions
RUN chown -R www-data:www-data /var/www/application  ->MUDAR AQUI 
RUN chmod -R 777 /var/www/application  ->MUDAR AQUI  

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]

## ----------------------------------------------------

version: "3.8"

services:
  app:
    build:
      context: ./
      dockerfile: docker/php/DockerFile
    container_name: dewtech-app
    restart: always
    working_dir: /var/www/
    volumes:
      - ./:/var/www
      - ./application:/var/www/application -> MUDDAR AQUI 
    environment:
      - COMPOSER_HOME=/composer
      - COMPOSER_ALLOW_SUPERUSER=1
      - APP_ENV=local
      - APP_KEY=base64:V9lMOMKFhL7IdZ9/3KcjtT24SIqRw+Oz7ObmYy9YvJQ=
    depends_on:
      - db
      - redis

  nginx:
    image: nginx:1.25.1-alpine-slim
    container_name: dewtech-nginx
    restart: always
    ports:
      - "8000:80"
    volumes:
      - ./:/var/www
      - ./application:/var/www/application -> MUDAR AQUI 
      - ./docker/nginx:/etc/nginx/conf.d

  redis:
    build:
      context: ./
      dockerfile: docker/redis/DockerFile
    container_name: dewtech-redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis:/data

  db:
    build:
      context: ./
      dockerfile: docker/db/DockerFile
    container_name: dewtech-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: your-root-password
      MYSQL_DATABASE: your-database-name
      MYSQL_USER: your-username
      MYSQL_PASSWORD: your-password
    ports:
      - "3306:3306"
    volumes:
      - db:/var/lib/mysql

volumes:
  redis:
  db:




