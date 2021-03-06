# Konfiguracja Xdebuga (macOS, PhpStorm, Docker, Symfony)

# Docker:

## docker/Dockerfile
```dockerfile
FROM php:8.0-fpm

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
    curl

RUN apt-get update && apt-get install -y procps && rm -rf /var/lib/apt/lists/*

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# XDEBUG
RUN pecl install xdebug-3.0.3
RUN docker-php-ext-enable xdebug

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```

## docker/default.conf

```
server {
    index index.php index.html;
    server_name test.local;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/html/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

## docker/conf.d/xdebug.ini

```
zend_extension=xdebug

[xdebug]
xdebug.mode=develop,debug
xdebug.client_host=host.docker.internal
xdebug.client_port=9003
xdebug.start_with_request=yes
xdebug.max_nesting_level=1500
```

## docker/conf.d/error_reporting.ini

```
error_reporting=E_ALL
```

## docker-compose.yaml

```
version: '3'

services:
    web:
        image: nginx:latest
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - ./:/var/www/html
            - ./docker/default.conf:/etc/nginx/conf.d/default.conf
        networks:
            - super-umbrella
    php-fpm:
        build:
            context: .
            dockerfile: docker/Dockerfile
        volumes:
            - ./:/var/www/html
            - ./docker/conf.d/xdebug.ini:/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
            - ./docker/conf.d/error_reporting.ini:/usr/local/etc/php/conf.d/error_reporting.ini
        networks:
            - super-umbrella

networks:
    super-umbrella:
        driver: bridge
```

# Konfiguracja PhpStorm

1. Ustaw interpreter
    ![](./resources/1.png)
    ![](./resources/2.png)

2. Upewnij si??, ??e ??cie??ki mapowania mi??dzy kontenerem a lokalnymi plikami s?? poprawne
    ![](./resources/3.png)

3. Ustaw konfiguracj?? Debuggera
    ![](./resources/10.png)
    ![](./resources/11.png)

4. Ustaw konfiguracj?? Serwera (upewnij si?? ??e ??cie??ki s?? zmapowane poprawnie)
    ![](./resources/18.png)

5. Ustaw konfiguracj?? Composera
    ![](./resources/4.png)

6. Ustaw konfiguracj?? PhpUnita
    ![](./resources/5.png)
    ![](./resources/6.png)
    ![](./resources/7.png)

7. Mo??na zrestartowa?? PHPStorma i odpali?? Dockera po restarcie

8. Mo??na odpala?? testy z poziomu IDE
    ![](./resources/8.png)
9. Mo??na odpala?? testy z poziomu IDE razem z Coverage
    ![](./resources/9.png)
10. Mo??na odpala?? debugowanie z poziomu test??w
    ![](./resources/12.png)
    ![](./resources/13.png)
11. Naciskamy ikonk?? z "Start Listening for PHP Debug Connection" - prawy g??rny r??g
    ![](./resources/14.png)
12. Odpalamy w przegl??darce aplikacj?? (zgodnie z tym co wpisali??my w pkt. 4)
    ![](./resources/19.png)
13. Wywalamy nasze `xdebug_info()` i wracamy do pierwotnej wersji
    ![](./resources/15.png)
14. Stawiamy kropy, od??wie??amy stron??, w PhpStormie powinna nam si?? odpali?? zak??adka z Debbugerem 
(aby przej???? do kolejnego breakpointa - fachowa nazwa na czerwon?? krop?? - klikamy na zielon?? strza??k?? - pierwsza po lewej w okienku Debuggera)
    ![](./resources/16.png)
15. Jak dotrzemy do ko??ca (klikaj??c zielon?? strza??k??) to w ko??cu pozwolimy na wykonanie requestu do ko??ca
    ![](./resources/17.png)
16. Mo??emy testowa?? r??wnie?? z poziomu klienta http wbudowanego w PhpStorma
17. Odpalamy dokumentacj?? w formacie np. OpenApi
    ![](./resources/20.png)
18. Ustawiamy autoryzacj?? (Authorize)
    ![](./resources/21.png)
    ![](./resources/22.png)
19. Generujemy cURLa requestu na podstawie dokumentacji OpenApi (Try it out -> Execute)
    ![](./resources/23.png)
    ![](./resources/24.png)
    ![](./resources/25.png)
    ![](./resources/26.png)
20. Konwertujemy cURLa na format klienta http PhpStorma
    ![](./resources/27.png)
    ![](./resources/28.png)
21. Je??li chemy odpali?? request trzeba wy????czy?? nas??uchiwanie Xdebuga i odpali?? request
    ![](./resources/29.png)
    ![](./resources/30.png)
22. Je??li chemy odpali?? debugger z requestu trzeba w????czy?? nas??uchiwanie Xdebuga, 
odpali?? request, odpali si?? okienko Debuggera i leci z koksem (zielona strza??ka -> Resume program)
    ![](./resources/31.png)
    ![](./resources/32.png)
    ![](./resources/33.png)
    ![](./resources/34.png)


## Je??li docker skonfigurowany poza projektem 

1. Trzeba doinstalowa?? tam Xdebuga 
2. Skonfigurowa?? cz????c rzeczy w PhpStormie jak poni??ej (reszta tak samo)

![](./resources/35.png)
![](./resources/36.png)
![](./resources/37.png)
![](./resources/38.png)



