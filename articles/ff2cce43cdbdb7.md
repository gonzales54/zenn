---
title: "DockerでLaravel9(Lemp)の環境構築"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# DockerでLaravel9(Lemp)の環境構築の手順のメモです。

## 構築完了後のディレクトリ構成
├── docker
│   ├── 1. app     
│   ├── 2. db      
│   └── 3. web     
└── 4. src        
1 php関連
2 mysql関連
3 nginx関連
4 Laravelの生成されたファイル、フォルダが入ります

### 1 適当なファルダに入り、下記を実行
```
touch docker-compose.yml
mkdir -p docker/app
mkdir -p docker/db
mkdir -p docker/web
```

### 2 docker-compose.ymlを書く
::: details docker-compose.yml
``` docker-compose.yml
version: "3.9"

services:
  app: 
    build:
      context: .
      dockerfile: ./docker/app/Dockerfile
    volumes:
      - ./src/:/app
      
  web:
    build:
      context: .
      dockerfile: ./docker/web/Dockerfile
    ports:
      - 8080:80
    depends_on:
      - app
    volumes:
      - ./src/:/app
  
  db:  
    build:
      context: .
      dockerfile: ./docker/db/Dockerfile
    ports:
      - 3306:3306
    environment:
      MYSQL_DATABASE: database
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: password
      TZ: 'Asia/Tokyo'
    volumes:
      - mysql-volume:/var/lib/mysql

volumes:
  mysql-volume:

```
:::
docker内のディレクトリは/appとします。
volumes:
  mysql-volume:
とすることでコンテナが停止してもデータが永続化される


### 3 dockerディレクトリ以下のファイルを書く

::: details docker/app/Dockerfile
``` docker/app/Dockerfile
FROM php:8.1-fpm

ENV TZ Asia/Tokyo

RUN apt-get update && \
	apt-get install -y git unzip libzip-dev libicu-dev libonig-dev && \
	docker-php-ext-install intl pdo_mysql zip bcmath
		
COPY ./docker/app/php.ini /usr/local/etc/php/php.ini

COPY --from=composer:2.0 /usr/bin/composer /usr/bin/composer

RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
RUN apt update && apt install nodejs

WORKDIR /app

```
:::

::: details docker/app/php.ini
``` docker/app/php.ini
zend.exception_ignore_args = off
expose_php = on
max_execution_time = 30
max_input_vars = 1000
upload_max_filesize = 64M
post_max_size = 128M
memory_limit = 256M
error_reporting = E_ALL
display_errors = on
display_startup_errors = on
log_errors = on
error_log = /var/log/php/php-error.log
default_charset = UTF-8

[Date]
date.timezone = Asia/Tokyo

[mysqlnd]
mysqlnd.collect_memory_statistics = on

[Assertion]
zend.assertions = 1

[mbstring]
mbstring.language = Japanese

```
:::
::: details docker/db/Dockerfile
``` docker/db/Dockerfile
FROM mysql:8.0

COPY ./docker/db/my.conf /etc/my.conf

```
:::
::: details docker/db/my.conf
``` docker/db/my.conf
[mysqld]
# character
character_set_server = utf8mb4
collation_server = utf8mb4_0900_ai_ci

# timezone
default-time-zone = SYSTEM
log_timestamps = SYSTEM

# Error Log
log-error = mysql-error.log

# Slow Query Log
slow_query_log = 1
slow_query_log_file = mysql-slow.log
long_query_time = 1.0
log_queries_not_using_indexes = 0

# General Log
general_log = 1
general_log_file = mysql-general.log

[mysql]
default-character-set = utf8mb4

[client]
default-character-set = utf8mb4

```
:::

::: details docker/web/default.conf
``` docker/web/default.conf
server {
    listen 80;
    server_name example.com;
    root /app/public; #appはdocker-compose.ymlのservices以下に記述したphpのコンテナ名

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass app:9000; #appはdocker-compose.ymlのservices以下に記述したphpのコンテナ名
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```
:::

::: details docker/web/Dockerfile
``` docker/web/Dockerfile
FROM nginx:1.20-alpine

ENV TZ Asia/Tokyo

COPY ./docker/web/default.conf /etc/nginx/conf.d/default.conf

```
:::


### 4 docker-composeで起動
``` command
docker-compose up -d --build
docker-compose ps
```
2個目のコマンドでちゃんと起動されているか確認(exited (1)となっている場合エラーが発生しています。)
![altテキスト](/images/docker-ps.png)

### 5 mysqlの接続確認
``` command
docker-compose exec db bash
mysql -u user -p #docker-compose.ymlで作成したユーザーとパスワードでログインできるか確認
Enter password
exit
```

### 6 laravelアプリの作成
``` command
docker-compose exec app bash
コンテナ内に入って
composer create-project "laravel/laravel=" . --prefer-dist //私はカレントディレクトリに作りました。
```
.envを編集
::: details .env
``` .env
DB_CONNECTION=mysql
DB_HOST=db  //dbはdocker-compose.ymlにあるmysqlのコンテナ名です
DB_PORT=3306
DB_DATABASE=database  //dbのデータベース名
DB_USERNAME=user      //dbのユーザー名
DB_PASSWORD=password  //dbのパスワード
```
:::

``` command
php artisan migrate
```

結果
![alt](/images/laravel-open.png)