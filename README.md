# A local development tool powered with Docker

This tool contains a web server with nginx, php7+ and MariaDB for docker compose.

For macOS and documentation describing how to install and use [Docker](http://docker.com). It doesn't explain what Docker is and so for more information
about that, read the Docker **[docs](https://docs.docker.com/)**.

## Install Docker

It's now possible to install Docker directly on your Mac.

1. Download the application "[Docker for Mac](https://www.docker.com/products/docker#/mac)"
2. Unpack the app and start the installation.

**After the application start, you can edit these preferences:**

* Increase to `4Cpus` and `4Gb` (or more) in General tab
* and `uncheck` "Automatic Update" to avoid problems with a broken release.
* Also `uncheck` usage and crash report in the Privacy tab.

At this point, a Docker deamon is started on your machine and you can
deploy container directly on it.

> You can run a simple test with `docker run hello-world`
> Then, run `docker ps -a` to see the container list
> To remove this useless container run `docker rm hello-world`

## The basic commands

To manage your containers, you normally use the `docker` command;
But this command (and all his parameters) is used to control one container at a time.

In this project, we want to use multiple containers:

* for web service
* database ...etc

So, use `docker-compose`...

> It's important to know how this command works so, one more time, read the
 **[docs](https://docs.docker.com/engine/reference/commandline/#the-docker-commands)**
 on the commands.

## Add to your project

If you respect the _structure_, using this tool is extremely simple:

1. Create a `docker-compose.yml` (see below) in your `root` project.
2. Start docker with `docker-compose up -d`

**When you finish your work on project:**

1. Simply stop the containers with `docker-compose down` OR `docker stop $(docker ps -q)` (if you're outside the project folder).

> These commands are really simplified. For more information, read the
 **[docs](https://docs.docker.com/engine/reference/commandline/#the-docker-commands)**
 on the commands.

## Using Docker (compose), for every day

We use `docker-compose` to create the environment for our projects, a utility
provided by docker to prepare containers and those interactions (Ports, Volumes).

#### `docker-compose.yml`:

You can copy it in your project root folder.

```
version: "3"

services:
  database:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: docker
      MYSQL_DATABASE: docker_dev
    restart: always
    ports:
      - "3306:3306"
    volumes:
      - "./database/dump:/docker-entrypoint-initdb.d"

  php:
    build: provision/php
    depends_on:
      - database
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - "./www:/var/www/html/"
      - "./database:/var/www/database"

  web:
    build: provision/web
    depends_on:
      - php
    links:
      - php
      - database
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "./www:/var/www/html/"
      - "./database:/var/www/database"
```

Here we define 3 containers :

* One for `database` using _MariaDB_ image
* One for `php` using the _PHP-FPM_ in version 7
* One for `web` with PHP processing using _Nginx_

In the last two, we set 2 folders as volumes for: `www` and `database`.

> For more information look at the **[docs](https://docs.docker.com/compose/overview/)**
 on compose.

#### Setup (provision):

The directory containing the configurations required for `docker-compose`
to set up the `php` and `web` containers.

### Start / Stop

* Go to the root of your project and `docker-compose build` to create to container images.
* then use `docker-compose up -d` to start your docker project (in background with `-d` option)
* You can type `docker logs -f` to show the flow of logs during your work session
* and `docker-compose down` to stop the containers.

The first time you start a `docker-compose` for your project, it can take a
while because docker needs to download the images and build the containers.

### Connect

By default this tool will be served on the address: [localhost](http://localhost)

#### Connect the database

* _MYSQL Database Host_: `database`
* _MYSQL Database_: `docker_dev`
* _MYSQL Root User_: `root`
* _MYSQL Root Password_: `docker`

_OR_ edit the `docker-compose.yml` with your desired configuration.

Because we are using **Docker for Mac**, the database is available
at localhost:3306 for applications like _Sequel Pro_.

#### Database dumps

Using the command:

`docker exec mariadb-container sh -c 'exec mysqldump --all-databases -uroot -p "$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql`

The command to create a dump for database docker_dev is:

`docker exec mariadb-container sh -c 'exec mysqldump --databases "docker_dev" -uroot -p "docker"' > /some/path/on/your/host/database.sql`

#### Persistent Database data

For a persistent database edit the `docker-compose.yml` and amend with the following:

```
volumes:
  - "./database/dump:/docker-entrypoint-initdb.d"
  - "./database/db:/var/lib/mysql"

```

#### Shell access

To run a shell command on a specific container:

`docker exec -t -i CONTAINER /bin/bash`

A good example would be to run the shell command on the `php` container
created by this project. Allowing access to run `composer` to install any
composer php dependencies.
