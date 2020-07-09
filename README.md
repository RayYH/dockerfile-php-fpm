# Laravel Application Example

## `docker-compose.yml`

```yml
version: "3"
services:
  nginx:
    image: nginx:1.18.0-alpine
    hostname: nginx
    networks:
      production:
        aliases:
          - nginx.production
    ports:
      - 80:80/tcp
    volumes:
      - the/path/to/nginx/default.d:/etc/nginx/default.d:rw
      - /the/path/to/nginx/conf.d:/etc/nginx/conf.d:rw
      - /the/path/to/nginx.conf:/etc/nginx/nginx.conf:rw
      - /the/path/to/app:/var/www/app:rw
    restart: always
  app:
    image: rayyounghong/php-fpm:latest
    hostname: app.production
    networks:
      production:
        aliases:
          - app.production
    volumes:
      - /the/path/to/app:/var/www/app:rw
    restart: always
  maraidb:
    image: mariadb:10.4
    hostname: mariadb.production
    networks:
      production:
        aliases:
          - mariadb.production
    environment:
      MYSQL_ROOT_PASSWORD: ROOT_PASSWORD
      MYSQL_USER: APP_NAME
      MYSQL_PASSWORD: STRONG_PASSWORD
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/log/mysql:/var/log/mysql:rw
      - /the/path/to/mariadb/conf.d:/etc/mysql/mariadb.conf.d:rw
      - mariadb:/var/lib/mysql
    restart: always
  redis:
    image: redis:latest
    hostname: redis.production
    networks:
      production:
        aliases:
          - redis.production
    volumes:
      - /etc/localtime:/etc/localtime:ro
    restart: always
volumes:
  mariadb:
    external: true
networks:
  production:
    external:
      name: production
```

## `nginx.conf`

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /var/www/app/app/public;
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        root           /var/www/app/app/public;
        fastcgi_pass   app.production:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```