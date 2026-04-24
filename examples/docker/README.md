# Run librebooking with docker

## Using the command line

This example features:

* A librebooking container reachable at <http://localhost:8080>
* A cron container executing scheduled librebooking-related jobs
* A database container hosting the librebooking data
* Persistent volumes storage for the database, librebooking configuration,
uploaded images and reservations

Adapt files `db.env`and `lb.env` to your needs

Create a container network

```sh
docker network create librebooking
```

Start the database container

```sh
docker container run \
  --name librebooking-db \
  --detach \
  --network librebooking \
  --hostname db \
  --volume librebooking-db_conf:/config \
  --env-file db.env \
  docker.io/linuxserver/mariadb:10.6.13
```

Start the application container

```sh
docker run \
  --name librebooking-app \
  --detach \
  --network librebooking \
  --publish 8080:8080 \
  --volume ./crontab:/config/lb-jobs-cron \
  --volume librebooking-app_conf:/config \
  --volume librebooking-app_img:/var/www/html/Web/uploads/images \
  --volume librebooking-app_res:/var/www/html/Web/uploads/reservation \
  --env-file lb.env \
 docker.io/librebooking/librebooking:develop
```

Start the cron container

```sh
docker run \
  --name librebooking-cron \
  --detach \
  --network librebooking \
  --volumes-from librebooking-app \
  --env-file lb.env \
  docker.io/librebooking/librebooking:develop \
  supercronic /config/lb-jobs-cron
```

## Using docker compose

This example features:

* A librebooking container reachable at <http://localhost:8080>
* A cron container executing scheduled librebooking-related jobs
* A database container hosting the librebooking data
* Persistent volumes storage for the database, librebooking configuration,
uploaded images and reservations

Adapt files `db.env`and `lb.env` to your needs

Start the application

```sh
docker compose --file docker-compose-local.yml up --detach
```

## Using docker compose (Internet)

This example features:

* A reverse proxy based on nginx that automatically handle certificates
* A librebooking container reachable at <https://your.host.com>
* A cron container executing scheduled librebooking-related jobs
* The usage of secrets to store passwords

Adapt files `db.env`and `lb.env` to your needs

Start the application

```sh
docker compose --file docker-compose-internet.yml up --detach
```
