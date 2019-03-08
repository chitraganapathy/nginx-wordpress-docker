
The [official WordPress image](https://hub.docker.com/_/wordpress/) has a PHP-FPM variant, but it still needs a web server to handle HTTP requests. This image provides an Nginx server ready to use as a `wordpress:fpm` front-end.

The Nginx configuration in this image is based on the guidelines given by the [Wordpress Codex](https://codex.wordpress.org/Nginx).

### How to Use This Image

    $ docker run --name some-nginx --link some-wordpress:wordpress --volumes-from some-wordpress -d -p 8080:80 raulr/nginx-wordpress

#### Environment Variables

* `POST_MAX_SIZE`: Sets max size of post data allowed. Also affects file uploads (defaults to `64m`).
* `BEHIND_PROXY`: Set to `true` if this container is behind a reverse proxy (defaults to `false` unless `VIRTUAL_HOST` environment variable is set).
* `REAL_IP_HEADER`: Defines the request header that will be used to obtain the real client IP when `BEHIND_PROXY` is set to `true` (defaults to `X-Forwarded-For`).
* `REAL_IP_FROM`: Defines trusted addresses to obtain the real client IP when `BEHIND_PROXY` is set to `true` (defaults to `172.16.0.0/12`).
* `WP_CONTAINER_NAME`: Defines your Wordpress (PHP-FPM) container's name aka fastcgi_pass (defaults to `wordpress`).

### ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml`:

```yaml
version: '3.1'

services:

  wordpress:
    image: wordpress:5.1.0-php7.3-fpm
    restart: always
    ports:
      - 9000:9000
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: example_dev
      WORDPRESS_DB_PASSWORD: example
      WORDPRESS_DB_NAME: example_dev
    volumes:
      - wp-data:/var/www/html
  nginx:
    image: raulr/nginx-wordpress
    links:
      - wordpress
    ports:
      - "80:80"
    environment:
      POST_MAX_SIZE: 128m
    volumes:
      - wp-data:/var/www/html
  
  db:
    image: mariadb:10.3
    restart: always
    environment:
      MYSQL_DATABASE: example_dev
      MYSQL_USER: example_dev
      MYSQL_PASSWORD: example
      MYSQL_ROOT_PASSWORD: example

volumes:
  wp-data:
```

Run `docker-compose up`, wait for it to initialize completely, and visit `http://localhost:8080` or `http://host-ip:8080`.
