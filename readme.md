# DAMP - Docker Apache MariaDb Php

## Basic Apache+MariaDb+php development environment.

This is a docker-compose file to quickly set up a development environment with apache, mariadb and php on docker.

> Think of Docker as a virtual machine, that runs a minimal Linux system with only the bare minimum required for your project, encapsulating all the necessary components in **containers**. You choose and set up the containers using **dockerfiles**, which contain the commands to build the container. To make life easier, docker-compose is a tool which enables you to describe the containers in a bit simpler **yaml** format. This project here is an example of how to set up a basic Apache, MariaDb, PHP development environment.
> In order to run this development environment, you will need to have docker and docker-compose installed on your system.
>
> *Optionally Docker desktop is a GUI app for managing your docker containers.*

## Usage
- Copy the contents of this repository to your project directory, open up a console in said directory and run: `docker-compose up`
- The contents of the /htdocs foler will be mapped to Apache's document root and show up at localhost:8080
- /mysql_data folder will map to /var/lib/mysql inside the mariadb container and hold MariaDb's data
- You also get PhpMyAdmin running on localhost:8081
- If you make any changes to the docker-compose or the dockerfile along the way, run `docker-compose up --build` to rebuild the container.


### Overview of the docker-compose file and what each line does.

``` yaml
version: '3' 
services:

    apache: #apache service
        build: # Specifies how to build the image for this service.
          # Since we're adding modules to php, we can't just use a vanilla image here
          # But instead we'll make a dockerfile, that lists the base image and instructions
          # To run after the image has been pulled to add extensions.
          # See inside phpapache.dockerfile
          context: . # Specifies the build context to use, in this case the current directory.
          dockerfile: phpapache.dockerfile # Specifies the Dockerfile to use for building the image.
        ports:
          - "8080:80" # Maps port 8080 on the host to port 80 inside the container - where apache is running.
        volumes:
          - ./htdocs:/var/www/html # Maps the local ./htdocs directory to /var/www/html inside the container.
          - ./logs:/var/log # Maps the local ./logs directory to /var/log inside the container.

          # To make it so Apache has full read and write access to the local /htdocs directory
          # We make Apache run as root.
          #
          # The APACHE_RUN_USER environment variable specifies the user under which the Apache web server should run inside the container.
          #
          # The value 1000 is the default user ID for the first user created on a Linux system = root
          # Normally Apache runs under the www-data user, so setting APACHE_RUN_USER to 1000
          # Makes Apache run as root, taking care of any file writing permissions with one dirty line~
          # Keep in mind THIS IS MEANT TO BE RAN AS A LOCAL DEV ENVIRONMENT.

        environment:
          - APACHE_RUN_USER=#1000
          - APACHE_RUN_GROUP=#1000
        depends_on: # Specifies the services that this service depends on.
          - mysql # Docker will wait for the 'mysql' service to start before starting the 'apache' service.

    mysql:
      image: mariadb:latest # Specifies the image to use for this service.
      environment:
          MYSQL_ROOT_PASSWORD: 'secret' # MariaDb root password
          MYSQL_USER: 'user'            # An additional user for mariadb
          MYSQL_PASSWORD: 'secret'      # Password for this additional user
          MYSQL_DATABASE: 'db1'         # Default database name
      volumes:
        - ./mysql_data:/var/lib/mysql   # Maps the local /mysql_data folder to /var_lib/mysql inside the contaier = the fodler mariadb keeps it's database data in.
      ports:
        - "3306:3306"                   # Mariadb ports 

    phpmyadmin:                         # pehMyAdmin service. Remove this section if you don't need it.
        image: phpmyadmin/phpmyadmin
        environment:
            PMA_HOST: mysql             # Host name for the sql server
            PMA_USER: user              # Username for the sql server
            PMA_PASSWORD: secret        # Password for the sql server
        ports:
            - "8081:80"                 # Maps host machine's port 8081 to port 80 inside the service container. 
        depends_on:
            - mysql                     # Wont start the service before 'mysql' service is up.
```

### Overview of the phpapache.dockerfile
This file contains the commands to build the apache service container and is needed if you want to make changes to a base container - in this case, to install additional extensions for php.

```dockerfile
# The base image to base the apache and php server on
# For a different php version, change this to something like ~ 'FROM php:8.1-apache'
FROM php:7.4-apache

# Enable mysqli pdo and pdo_mysql extensons for php
# Add any other extensions here you need to enable
# Either on the same line or as a new 'RUN docker-php-ext-install [extension]' line
RUN docker-php-ext-install mysqli pdo pdo_mysql
```