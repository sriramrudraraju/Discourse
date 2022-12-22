# Discourse (Bitnami Docker Images)

## Getting Started
* [bitnami/discourse](https://hub.docker.com/r/bitnami/discourse)
* Get the compose file and run the docker compose
```
$ curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/discourse/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```
* From docker compose
```docker
 discourse:
    .
    ports:
      - '80:3000' # Will be exposed to host port 80
    .
    environment:
      .
      - DISCOURSE_HOST=www.example.com # need to change this to localhost to verify locally
```
```diff
 discourse:
    .
    environment:
    .
-   - DISCOURSE_HOST=www.example.com
+   - DISCOURSE_HOST=localhost
    .
sidekiq:
    .
    environment:
      .
-     - DISCOURSE_HOST=www.example.com
+     - DISCOURSE_HOST=localhost
      .
```
* Discourse website can be seen at `localhost:80`
> Note: Since ports are not exposed to host, We cant look into databases. Expose the ports in compose file if needed for database connections.

## External Databases

* Official docker-compose creates internal database containers within docker network.

>Info: Will try to keep the same database names used in bitnami for less code changes.

### Connecting to external Postgres

* Create a postgres database with with docker at port `5432` add respective ENV variables for below details 

| Property | value |
| --- | --- |
| Database Name | bitnami_discourse |
| Database User | bn_discourse |
| Database Password | password |

* Check if database is created as expected using a GUI client.
* Once database is created update the docker-compose file.

```diff
version: '2'
services:
-  postgresql:
-    image: docker.io/bitnami/postgresql:15
-    volumes:
-      - 'postgresql_data:/bitnami/postgresql'
-    environment:
-      # ALLOW_EMPTY_PASSWORD is recommended only for development.
-      - ALLOW_EMPTY_PASSWORD=yes
-      - POSTGRESQL_USERNAME=bn_discourse
-      - POSTGRESQL_DATABASE=bitnami_discourse
  redis:
    image: docker.io/bitnami/redis:7.0
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - 'redis_data:/bitnami/redis'
  discourse:
    image: docker.io/bitnami/discourse:2
    ports:
      - '80:3000'
    volumes:
      - 'discourse_data:/bitnami/discourse'
    depends_on:
-      - postgresql
      - redis
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
      - DISCOURSE_HOST=localhost
-      - DISCOURSE_DATABASE_HOST=postgresql
+      - DISCOURSE_DATABASE_HOST=<local ip address> #localhost didnt work me :(
      - DISCOURSE_DATABASE_PORT_NUMBER=5432
      - DISCOURSE_DATABASE_USER=bn_discourse
      - DISCOURSE_DATABASE_NAME=bitnami_discourse
+      - DISCOURSE_DATABASE_PASSWORD=password
      - DISCOURSE_REDIS_HOST=redis
      - DISCOURSE_REDIS_PORT_NUMBER=6379
+      - POSTGRESQL_CLIENT_DATABASE_HOST=<local ip address> #localhost didnt work me :(
+      - POSTGRESQL_CLIENT_DATABASE_PORT_NUMBER=5432
-      - POSTGRESQL_CLIENT_POSTGRES_USER=postgres
+      - POSTGRESQL_CLIENT_POSTGRES_USER=postgres
+      - POSTGRESQL_CLIENT_POSTGRES_PASSWORD=password
      - POSTGRESQL_CLIENT_CREATE_DATABASE_NAME=bitnami_discourse
      - POSTGRESQL_CLIENT_CREATE_DATABASE_EXTENSIONS=hstore,pg_trgm
  sidekiq:
    image: docker.io/bitnami/discourse:2
    depends_on:
      - discourse
    volumes:
      - 'sidekiq_data:/bitnami/discourse'
    command: /opt/bitnami/scripts/discourse-sidekiq/run.sh
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
      - DISCOURSE_HOST=localhost
-      - DISCOURSE_DATABASE_HOST=postgresql
+      - DISCOURSE_DATABASE_HOST=<local ip address> #localhost didnt work me :(
      - DISCOURSE_DATABASE_PORT_NUMBER=5432
      - DISCOURSE_DATABASE_USER=bn_discourse
      - DISCOURSE_DATABASE_NAME=bitnami_discourse
+      - DISCOURSE_DATABASE_PASSWORD=password
      - DISCOURSE_REDIS_HOST=redis
      - DISCOURSE_REDIS_PORT_NUMBER=6379
volumes:
-  postgresql_data:
-    driver: local
  redis_data:
    driver: local
  discourse_data:
    driver: local
  sidekiq_data:
    driver: local
```

### Connecting to external Redis

* Create a redis database with docker image at port `6379`
* Once database is created, update docker-compose

```diff
version: '2'
services:
-  redis:
-    image: docker.io/bitnami/redis:7.0
-    environment:
-      # ALLOW_EMPTY_PASSWORD is recommended only for development.
-      - ALLOW_EMPTY_PASSWORD=yes
-    volumes:
-      - 'redis_data:/bitnami/redis'
  discourse:
    image: docker.io/bitnami/discourse:2
    ports:
      - '80:3000'
    volumes:
      - 'discourse_data:/bitnami/discourse'
-    depends_on:
-      - redis
    environment:
-      # ALLOW_EMPTY_PASSWORD is recommended only for development.
-      - ALLOW_EMPTY_PASSWORD=yes
      - DISCOURSE_HOST=localhost
      - DISCOURSE_DATABASE_HOST=<local ip address> #localhost didnt work me :(
      - DISCOURSE_DATABASE_PORT_NUMBER=5432
      - DISCOURSE_DATABASE_USER=bn_discourse
      - DISCOURSE_DATABASE_NAME=bitnami_discourse
      - DISCOURSE_DATABASE_PASSWORD=password
-      - DISCOURSE_REDIS_HOST=redis
+      - DISCOURSE_REDIS_HOST=<local ip address> #localhost didnt work me :(
      - DISCOURSE_REDIS_PORT_NUMBER=6379
      - POSTGRESQL_CLIENT_DATABASE_HOST=<local ip address> #localhost didnt work me :(
      - POSTGRESQL_CLIENT_DATABASE_PORT_NUMBER=5432
      - POSTGRESQL_CLIENT_POSTGRES_USER=bn_discourse
      - POSTGRESQL_CLIENT_POSTGRES_PASSWORD=password
      - POSTGRESQL_CLIENT_CREATE_DATABASE_NAME=bitnami_discourse
      - POSTGRESQL_CLIENT_CREATE_DATABASE_EXTENSIONS=hstore,pg_trgm
  sidekiq:
    image: docker.io/bitnami/discourse:2
    depends_on:
      - discourse
    volumes:
      - 'sidekiq_data:/bitnami/discourse'
    command: /opt/bitnami/scripts/discourse-sidekiq/run.sh
    environment:
-      # ALLOW_EMPTY_PASSWORD is recommended only for development.
-      - ALLOW_EMPTY_PASSWORD=yes
      - DISCOURSE_HOST=localhost
      - DISCOURSE_DATABASE_HOST=<local ip address> #localhost didnt work me :(
      - DISCOURSE_DATABASE_PORT_NUMBER=5432
      - DISCOURSE_DATABASE_USER=bn_discourse
      - DISCOURSE_DATABASE_NAME=bitnami_discourse
      - DISCOURSE_DATABASE_PASSWORD=password
-      - DISCOURSE_REDIS_HOST=redis
+      - DISCOURSE_REDIS_HOST=<local ip address> #localhost didnt work me :(
      - DISCOURSE_REDIS_PORT_NUMBER=6379
volumes:
-  redis_data:
-    driver: local
  discourse_data:
    driver: local
  sidekiq_data:
    driver: local
```

## Setting up admin user
* Go to terminal of discourse image's container.

```
cd /opt/bitnami/discourse
RAILS_ENV=production bundle exec rake admin:create
```