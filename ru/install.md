---
title: Установка панели
layout: default
lang: ru
category: Общее
order: 21
---

* This will become a table of contents (this text will be scraped).
{:toc}

## Debian/Ubuntu

### Установка необходимых пакетов

```
sudo apt-get update
sudo apt-get install -y wget curl gnupg ca-certificates apt-transport-https lsb-release
```

### Добавление репозиториев GameAP

Скачайте и добавьте GPG ключ:
```bash
wget -O - http://packages.gameap.ru/gameap-rep.gpg.key | sudo apt-key add -
```

Этот ключ служит для проверки подленности пакетов в репозиториях GameAP.


Добавьте репозиторий:
```bash
sudo echo "deb http://packages.gameap.ru/debian/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/gameap.list
```


### Установка панели

После того, как добавили репозиторий, обновите список пакетов и установите панель следующими командами:

```bash
sudo apt-get update
sudo apt-get install gameap
```

Панель будет установлена в каталог `/var/www/gameap`

## Настройка веб-сервера

### Nginx

Пример конфигурации nginx:

```
server {
    listen       80;
    server_name  _;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;
    root /var/www/gameap/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;

        location = /index.php
        {
            # fastcgi_pass   php:9000;
            fastcgi_pass    unix:/var/run/php/php7.2-fpm.sock;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
Обратите внимание на параметры `server_name` и `root`, скорее всего у вас они другие и их нужно указать.
В `server_name` указывается хост сервера, например gameap.ru.
В `root` указывается путь к public файлам панели (там, где index.php)

### Apache

Пример конфигурации хоста Apache:

```
NameVirtualHost *:80
Listen 80
 
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/gameap/public
     
    <Directory /var/www/gameap/public>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
            Require all granted
    </Directory>
     
    LogLevel debug
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## Настройка базы данных

### MySQL

После установки MySQL создайте базу данных `gameap`, откройте `.env` файл и отредактируйте параметры:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=gameap
DB_USERNAME=gameap
DB_PASSWORD=secret
```

После найстройки параметров перейдите в каталог с панелью управления `/var/www/gameap` и обновите схему базы данных:
```bash
php artisan migrate --seed
```

### SQLite

Создайте пустой файл базы данных:
```
touch /var/www/gameap/database.sqlite
```

Откройте `.env` и отредактируйте параметры базы данных:
```
DB_CONNECTION=sqlite
DB_DATABASE=/var/www/gameap/database.sqlite
```

После найстройки параметров перейдите в каталог с панелью управления `/var/www/gameap` и обновите схему базы данных:
```bash
php artisan migrate --seed
```
