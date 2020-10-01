# nginx-php-fpm

<img src="https://www.milesweb.in/hosting-faqs/wp-content/uploads/2019/05/How-To-Deploy-a-PHP-Application-with-Kubernetes-on-Ubuntu-16.04.gif" alt="Alt Text" style="zoom:60%" />

<br>

### Versions/tags

|  `software`  |      `version`      |
| :----------: | :-----------------: |
| `nginx-more` |      `1.16.1`       |
|    `php`     |      `7.3.22`       |
|  `php-fpm`   | `7.3.22 (fpm-fcgi)` |
|   `mysql`    |      `5.7.31`       |

<br>

### Links related to this Project

[LARAVEL](https://github.com/laravel/laravel)

[Dockerhub](https://hub.docker.com/r/shashwot/php-fpm)

[Kubernetes](https://github.com/shashwot/Kubernetes)

<br>

# Project

```bash
$ cd /root/
$ git clone https://github.com/laravel/laravel.git laravel-app
$ cd laravel-app
$ mv .env.example .env
# Change the value of DB_HOST as per your requirement 
$ composer install
# OR
$ docker run --rm -v $(pwd):/app composer install
$ vi Dockerfile
```

<br>

# Dockerfile

```dockerfile
FROM php:7.3-fpm

# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libzip-dev \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
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

# Remove the Dockerfile from the build
RUN rm -rf Dockerfile

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```

<br>

# Docker image

```bash
$ docker build -t “shashwot/php-fpm” .
#Or use the image below
$ docker pull shashwot/php-fpm
$ docker pull shashwot/nginx-more
```

<br>

# NGINX configuration

```nginx
server {
  listen 80;
  index index.php index.html;
  error_log  /var/log/nginx/php-error.log;
  access_log /var/log/nginx/php-access.log;
  root /var/www/public;
  location ~ \.php$ {
      try_files $uri =404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      fastcgi_pass php-fpm:9000;
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
```

<br>

# PHP Commands

```bash
$ php artisan key:generate
$ php artisan config:cache
$ php artisan migrate
```

<br>

# Mysql Commands

```sql
$ mysql -u root -p
mysql> show databases;
mysql> create database laravel;
mysql> GRANT ALL ON laravel.* TO 'laraveluser'@'%' IDENTIFIED BY 'your_laravel_db_password';
mysql> FLUSH PRIVILEGES;
```

<br>

# Kubernetes

```bash
# For installations go to: https://github.com/shashwot/Kubernetes

$ git clone https://github.com/shashwot/nginx-php-fpm.git
$ cd k8s
$ kubectl apply -f .
```

<br>

# Output

![Alt text](Laravel.png?raw=true "Laravel")
