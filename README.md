# Customized (by bozzochet) version of Official Docker Image for Grav

I just took the official one and I substituted the `curl` command downloading Grav with a `git clone` (from https://github.com/bozzochet/grav) that download my entire website.

Then a simple call to a bash script (`install.sh`) deploy it completely.

I also added two crontab entries
* to pull from git every hour to update the website content
* to run a script (`install_plugins.sh`) every night to add new plugins added to the script (and updated by the pull from github half an hours earlier or in any previous hourly update)

The website is not planned to have 'persistency' so the comments below are not relevant (I kept just for inventory benefit). Once built (as described below or by simply downloading the image from DockerHub, https://hub.docker.com/r/bozzochet/docker-grav) I simply run with

```
docker run -d -p 8000:80 grav:latest
```

or

```
docker run -d -p 8000:80 bozzochet/docker-grav:latest
```

(after pulling from DockerHub with `docker pull bozzochet/docker-grav`) and the process runs in background.

I can connect to it by:

```
docker exec -it --user root <contained ID> /bin/bash
```

or

```
docker exec -it --user www-data <contained ID> /bin/bash
```

---

This currently is pretty minimal and uses:

* apache-2.4.38
* GD library
* Unzip library
* php7.4
* php7.4-opcache
* php7.4-acpu
* php7.4-yaml
* cron
* vim editor

## Persisting data

To save the Grav site data to the host file system (so that it persists even after the container has been removed), simply map the container's `/var/www/html` directory to a named Docker volume or to a directory on the host.

> If the mapped directory or named volume is empty, it will be automatically populated with a fresh install of Grav the first time that the container starts. However, once the directory/volume has been populated, the data will persist and will not be overwritten the next time the container starts.

## Building the image from Dockerfile

```
docker build -t grav:latest .
```

## Running Grav Image with Latest Grav + Admin:

```
docker run -p 8000:80 grav:latest
```

Point browser to `http://localhost:8000` and create user account...

## Running Grav Image with Latest Grav + Admin with a named volume (can be used in production)

```
docker run -d -p 8000:80 --restart always -v grav_data:/var/www/html grav:latest
```

## Running Grav Image with docker-compose and a volume mapped to a local directory

Running `docker-compose up -d` with the following docker-compose configuration will automatically build the Grav image (if the Dockerfile is in the same directory as the docker-compose.yml file). Then the Grav container will be started with all of the site data persisted to a named volume (stored in the `./grav` directory.

```.yml
volumes:
  grav-data:
    driver: local
    driver_opts:
      type: none
      device: $PWD/grav
      o: bind

services:
  grav:
    build: ./
    ports:
      - 8080:80
    volumes:
      - grav-data:/var/www/html
```
