# docker-compose up と docker-compose build --no-cache

- ```docker-compose up``` は、```docker-compose.yml```に定義されたサービスをビルド（必要な場合）し、コンテナを起動する。

- ```docker-compose build --no-cache```は、docker-compose.yml ファイルに定義されたサービスのイメージをキャッシュを使用せずにビルドする。

⚠️ docker-compose build --no-cacheはコンテナ起動までは行わない。

# docker-compose.yml

docker-compose.ymlの見方

```
version: '3'
services:                                                  // この時のサービスは,laravel.test、mysql,adminer,minioの4つサービスがビルドされる。
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.1    // ビルドする時に参照するファイルがどこにあるかを記載
            dockerfile: Dockerfile                         // ./vendor/laravel/sail/runtimes/8.1配下にあるDockerfileを元にビルドを行う。
            args:
                WWWGROUP: '${WWWGROUP}'
                NODE_VERSION: '18'
        image: sail-8.0/app
        ports:
            - '${APP_PORT:-80}:80'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mysql
            - minio
    mysql:
        image: 'mysql:8.0'
        ports:
            - '${FORWARD_DB_PORT:-3307}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
        volumes:
            - 'sailmysql:/var/lib/mysql'
            - './storage:/var/log/mysql'
        networks:
            - sail
        healthcheck:
            retries: 3
            timeout: 5s
        command: mysqld --general_log=1 --general_log_file=/var/log/mysql/mysql-query.log
    adminer:
        image: adminer
        depends_on:
            - mysql
        environment:
            ADMINER_PLUGINS: 'tables-filter dump-zip'
            ADMINER_DEFAULT_SERVER: 'mysql'
        networks:
            - sail
        ports:
            - 8080:8080
        restart: unless-stopped
    minio:
        image: 'minio/minio:latest'
        ports:
            - '${FORWARD_MINIO_PORT:-9000}:9000'
            - '${FORWARD_MINIO_CONSOLE_PORT:-8900}:8900'
        environment:
            MINIO_ROOT_USER: sail
            MINIO_ROOT_PASSWORD: password
        volumes:
            - 'sail-minio:/data/minio'
        networks:
            - sail
        command: 'minio server /data/minio --console-address ":8900"'
        healthcheck:
            test:
                - CMD
                - curl
                - '-f'
                - 'http://localhost:9000/minio/health/live'
            retries: 3
            timeout: 5s
networks:
    sail:
        driver: bridge
volumes:
    sailmysql:
        driver: local
    sail-mysql:
        driver: local
    sail-minio:
        driver: local

```
