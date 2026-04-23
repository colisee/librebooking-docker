# Run LibreBooking with Podman

## Using the command line

This example features:

* A librebooking container reachable at <http://localhost:8080>
* A cron container executing scheduled librebooking-related jobs
* A database container hosting the librebooking data
* Persistent volumes storage for the database, librebooking configuration,
uploaded images and reservations

Adapt files `db.env`and `lb.env` to your needs

Create a pod

```sh
podman pod create --publish 8080:8080 librebooking
```

Add the database container to the pod

```sh
podman container create \
  --name db \
  --replace \
  --pod librebooking \
  --volume librebooking-db_conf:/config:U \
  --env-file db.env \
  docker.io/linuxserver/mariadb:10.6.13
```

Add the application container to the pod

```sh
podman container create \
  --name app \
  --replace \
  --pod librebooking \
  --volume ./crontab:/config/lb-jobs-cron \
  --volume librebooking-app_conf:/config:U \
  --volume librebooking-app_img:/var/www/html/Web/uploads/images \
  --volume librebooking-app_res:/var/www/html/Web/uploads/reservation \
  --env-file lb.env \
  docker.io/librebooking/librebooking:develop
```

Add the cron container to the pod

```sh
podman container create \
  --name job \
  --replace \
  --pod librebooking \
  --volumes-from app \
  --env-file lb.env \
  docker.io/librebooking/librebooking:develop \
  supercronic /config/lb-jobs-cron
```

Start the pod

```sh
podman pod start librebooking
```

Stop the pod

```sh
podman pod stop librebooking
```

## Using a Kubernetes yaml file

This example features:

* A librebooking container reachable at <http://localhost:8080>
* A cron container executing scheduled librebooking-related jobs
* A database container hosting the librebooking data
* Persistent volumes storage for the database, librebooking configuration,
uploaded images and reservations

Adapt the above command line instructions to your needs

Generate a Kubernetes file

```sh
podman generate kube --filename librebooking.yml
```

Start the application

```sh
podman kube play librebooking.yml
```

Stop the application

```sh
podman kube down librebooking.yml
```

## Using systemd

The following examples work on systemd-based distributions only. 
They require the lingering option to be set for your user.

```sh
loginctl enable-linger $USER
```

### systemd and podman-kube@service template

This example features:

* A librebooking container reachable at <http://localhost:8080>
* A cron container executing scheduled librebooking-related jobs
* A database container hosting the librebooking data
* Persistent volumes storage for the database, librebooking configuration,
uploaded images and reservations

Adapt the above file `librebooking.env` to your needs

Enable and start the service

```sh
escaped=$(systemd-escape /path/to/librebooking.yml)
systemctl --user enable --now podman-kube@$escaped.service
```

### systemd and quadlets

This example requires podman version 5.6.0 or greater. It features:

* Librebooking reachable at <http://localhost:8080> with scheduled jobs
* A persistent storage for the database, the librebooking configuration files, images and reservation attachments

Adapt file `librebooking.env` to your needs

Build the image, installs the quadlet files into `~/.config/containers/systemd/`, and starts all services:

```sh
./scripts/start
```

Stop all services and removes the quadlet unit files:

```sh
./scripts/stop
```

Runs stop then start (rebuilds the image):

```sh
./scripts/restart
```

Monitor the jobs

```sh
systemctl --user status app db cron
journalctl --user -u app -f
journalctl --user -u cron -f
journalctl --user -u db -f
```

