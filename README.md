# sail-php-pgsql-memcached
Laravel Docker Sail compose stub for PHP 8.1, Postgres 13 and Memcached Server

Official Docs - https://laravel.com/docs/sail

**.env stub**

```
# DB 

DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=test
DB_USERNAME=test
DB_PASSWORD=test

## Memcached

MEMCACHED_HOST=memcached
CACHE_DRIVER=memcached

```

**docker-compose.yml**

```
version: '3'
services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.1
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.1/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - pgsql
            - redis
    pgsql:
        image: 'postgres:13'
        ports:
            - '${FORWARD_DB_PORT:-5432}:5432'
        environment:
            PGPASSWORD: '${DB_PASSWORD:-secret}'
            POSTGRES_DB: '${DB_DATABASE}'
            POSTGRES_USER: '${DB_USERNAME}'
            POSTGRES_PASSWORD: '${DB_PASSWORD:-secret}'
        volumes:
            - 'sailpgsql:/var/lib/postgresql/data'
        networks:
            - sail
        healthcheck:
            test: [ "CMD", "pg_isready", "-q", "-d", "${DB_DATABASE}", "-U", "${DB_USERNAME}" ]
            retries: 3
            timeout: 5s
    memcached:
        image: 'memcached:alpine'
        ports:
            - '11211:11211'
        networks:
            - sail
networks:
    sail:
        driver: bridge
volumes:
    sailpgsql:
        driver: local

```
