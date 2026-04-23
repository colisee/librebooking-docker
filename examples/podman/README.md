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

## Using quadlets

Unlike the previous examples, this setup requires a systemd-based operating
system. It features:

* Librebooking reachable at <http://localhost:8080> with scheduled jobs
* A persistent storage for the database, the librebooking configuration files, images and reservation attachments

Adapt file `librebooking.env` to your needs

Enable the lingering option for your user

```sh
loginctl enable-linger $USER
```

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

## Using systemd (production)

This setup is meant for accessing the application for production purposes.
[Automatic updates](https://docs.podman.io/en/latest/markdown/podman-auto-update.1.html)
for container images are not enabled in this example. Try it also, it's handy.

## Create network file

```sh
cat >> ~/.config/containers/systemd/librebooking.network<<EOF 
[Unit]
Description=Librebooking Network

[Network]
Subnet=192.168.30.0/24
Gateway=192.168.30.1
Label=app=librebooking
EOF
```

## create volume for DB

```sh
cat >> ~/.config/containers/systemd/mariadb-lb.volume<<EOF
[Volume]
Driver=local
Label=app=librebooking
EOF
```

## Create DB container conf

```sh
cat >> ~/.config/containers/systemd/mariadb-lb.container<<EOF
[Unit]
Description=MariaDB container

[Container]
Image=docker.io/linuxserver/mariadb:10.6.13
Environment=MYSQL_ROOT_PASSWORD=db_root_pwd
Environment=MYSQL_USER=lb
Environment=MYSQL_PASSWORD=lb-test
Environment=MYSQL_DATABASE=db
Environment=PUID=1000
Environment=PGID=1000
Volume=mariadb-lb.volume:/config:U
Network=librebooking.network
PublishPort=3306:3306
Label=app=librebooking
# AutoUpdate=registry

[Service]
Restart=on-failure
EOF
```

## Create Images Volume

```sh
cat >> ~/.config/containers/systemd/lb-images.volume<<EOF
[Volume]
Driver=local
Label=app=librebooking
EOF
```

## Create Reservations Volume

```sh
cat >> ~/.config/containers/systemd/lb-reservation.volume<<EOF
[Volume]
Driver=local
Label=app=librebooking
EOF
```

## Create LB container file

```sh
cat >> ~/.config/containers/systemd/lb.container<<EOF
[Unit]
Description=Librebooking container
Requires=mariadb-lb.service
After=mariadb-lb.service
 
[Container]
HostName=librebooking
Image=docker.io/librebooking/librebooking:develop
Network=librebooking.network
Environment=LB_DATABASE_NAME=db
Environment=LB_DATABASE_USER=lb
Environment=LB_DATABASE_PASSWORD=lb-test
Environment=LB_DATABASE_HOSTSPEC=systemd-mariadb-lb
Environment=LB_INSTALL_PASSWORD=installme
Environment=LB_LOGGING_FOLDER=/var/log/librebooking
Environment=LB_LOGGING_LEVEL=DEBUG
Environment=LB_LOGGING_SQL=false
Environment=LB_DEFAULT_TIMEZONE=Europe/Helsinki
PublishPort=8080:8080
Label=app=librebooking
Volume=lb-images.volume:/var/www/html/Web/uploads/images
Volume=lb-reservation.volume:/var/www/html/Web/uploads/reservation
Volume=%h/librebooking-conf:/config:U
# AutoUpdate=registry
# User=1000:1000


[Install]
WantedBy=multi-user.target

[Service]
Restart=on-failure
EOF
```

## Create permanent conf dir for LB

```sh
mkdir ~/librebooking-conf
```

## Start the services

Now we have all the config files done for systemd. Next we reload the daemon, and start the services:

```sh
systemctl --user daemon-reload
systemctl --user start mariadb-lb
systemctl --user start lb
```

## Enable autostart at boot

To make the systemd start the containers automatically at boot you need to
enable lingering the the user and enable the services:

```sh
sudo loginctl enable-linger $USER
systemctl --user enable --now mariadb-lb
systemctl --user enable --now lb
```

## Logs

If you want to see the logs, you can use `podman logs ...` or use `journalctl --user -u lb`. You can also modify the conf file in ~/librebooking-conf/config.php, and restart the service with `systemctl --user restart lb`.

# Connect to Librebooking

At this point the system is running at http://<hostname>:8080.

*Note: I tested the config on Fedora 43.*
