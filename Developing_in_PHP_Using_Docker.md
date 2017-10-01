# Developing in PHP Using Docker

## Step 1: Install Docker for Mac

Visit <https://www.docker.com/docker-mac> and download Docker for Mac.  It's free!

## Step 2: Build and Extract a Docker Compose File

Visit <https://phpdocker.io/generator>.  Fill in the following fields:

- Project Name - call it whatever you like.  Don't use spaces.  Example: ```docker-web```
- Base port:  Use something you're not already using on your local machine.  Example: ```8022```.  Whatever you use, you'll use later when you browse to your site, e.g. ```http://localhost:8022```
- Extensions: I usually select ```mysql``` and ```xdebug```.
- Turn on the switch for mysql (or MariaDB or Postgres) if you need a database.
  - If you do so, you'll be asked for account names and passwords.
  - root user pass: PestoMadnessFestival
  - app database name:  MyAwesomeDb 
  - app database username: victortequila
  - app database password: DropIcyBackpacks

- Click *Generate Project Archive*.  A zip file will download.
- Move the file to the location where you want the sandbox.  You do *not* need to create a new directory for it.  The root directory in the zip file will be whatever you named your project.  I moved mine into ~/Code
- Extract the zip file.

Here's what you'll have after you extract the file:

![Directory Structure](dir_structure.png)

### docker-compose.yml

docker-compose is a tool that lets you define multiple Docker containers in one file.  The contents of our docker-compose.yml file:

```
###############################################################################
#                          Generated on phpdocker.io                          #
###############################################################################
version: "3.1"
services:

    mysql:
      image: mysql:8.0
      container_name: docker-web-mysql
      working_dir: /application
      volumes:
        - .:/application
      environment:
        - MYSQL_ROOT_PASSWORD=PestoMadnessFestival
        - MYSQL_DATABASE=MyAwesomeDb
        - MYSQL_USER=victortequila
        - MYSQL_PASSWORD=DropIcyBackpacks
      ports:
        - "8024:3306"

    webserver:
      image: nginx:alpine
      container_name: docker-web-webserver
      working_dir: /application
      volumes:
          - .:/application
          - ./phpdocker/nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      ports:
       - "8022:80"

    php-fpm:
      build: phpdocker/php-fpm
      container_name: docker-web-php-fpm
      working_dir: /application
      volumes:
        - .:/application
        - ./phpdocker/php-fpm/php-ini-overrides.ini:/etc/php/7.1/fpm/conf.d/99-overrides.ini


```

We have 3 entries in the services section:  mysql, webserver, and php-fpm.  These are pretty self-explanatory:  one container is created for the database, one for the web server (nginx), and one for PHP.

**image** specifies the image to start the container from.  Since the image doesn't currently exist, Docker will pull it from a public repository.  Note that **php-fpm** has a **build** item instead.  In this case, a path is provided to a **build context**.  If you follow that directory path, you'll see a separate Dockerfile.

**container_name** allows you to override the container name that Docker would otherwise generate for you.

**working_dir** specifies the working directory of the container that is created.  In all 3 containers, the path is /application.

**volumes** specifies file and directory mappings from your machine (e.g. your Mac) to the container's file system. The line:

~~~
.:/application
~~~

is shorthand syntax for mapping the current directory on your Mac (where the docker-compose.yml is) to the directory /application in your container.

Note that the webserver container adds a second mapping: 

```
./phpdocker/nginx/nginx.conf:/etc/nginx/conf.d/default.conf
```

The nginx.conf file that you unzipped becomes the actual configuration file for the webserver.
