# Get or build the docker image

You have 2 ways to get the docker image on your host:

1. Get the image from the docker's hub
1. Build the image

## Get the image from the docker's hub

This is the easiest and fastest way.

- [Release](https://github.com/LibreBooking/librebooking/releases):

  ```sh
  docker image pull librebooking/librebooking
  ```

- On-going development:

  ```sh
  docker image pull librebooking/librebooking:nightly
  ```

## Build the docker image

1. Clone this repository

   ```sh
   git clone https://github.com/librebooking/docker
   ```

1. Decide which image you wish to buid:

   - [Release](https://github.com/LibreBooking/librebooking/releases):

     ```sh
     APP_GH_REF='v7.2.0'
     IMG_TAG=${APP_GH_REF#[vV]}
     APP_GH_ADD_SHA=false
     ```

   - On-going development:

     ```sh
     APP_GH_REF=''
     IMG_TAG=nightly
     APP_GH_ADD_SHA=true
     ```

1. Build the docker image (Docker 23+)

   ```sh
   docker build \
     --build-arg APP_GH_REF=${APP_GH_REF} \
     --build-arg APP_GH_ADD_SHA=${APP_GH_ADD_SHA} \
     --tag librebooking/librebooking:${IMG_TAG} \
     .
   ```

   Notes:
   
   - When `APP_GH_ADD_SHA=true`, the web UI footer appends the LibreBooking app
   short SHA to the displayed version string.
   - You can override additional build arguments if needed:
     + VERSION_PHP: php version
     + VERSION_COMPOSER: composer version
