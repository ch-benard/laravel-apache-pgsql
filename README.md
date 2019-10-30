# LARAVEL-APACHE2-PGSQL

Docker environment for a Laravel project with Apache2, PHP 7.3 and Postgresql 12

## Prerequisites

To use this stack, you need a recent version of git, docker and docker-compose that works on your computer. There are a lot of tutorial on the internet that can help you installing git, docker and docker-compose. I personally have these tools running on my PC that runs Ubuntu 18.04 and on my laptop with CloudReady, a nice ChromeOS fork.

## How to create a Laravel project from scratch with LARAVEL-APACHE2-PGSQL ?

1. Clone the laravel-apache2-pgsql repository

```bash
git clone https://github.com/ch-benard/laravel-apache-pgsql.git
```

2. Create the **postgresql-data** and **public_html** directories

```bash
cd laravel-apache-pgsql
mkdir postgresql-data public_html
```

3. Update your php **Dockerfile**

Once Docker is lauched, you'll be able to write your own files into your local **public_html** folder. This folder is mapped to the **/var/www/html** folder on the php container.
Every modification updates the web application which is running on your docker stack. So you can immediatly see the effect of your modifications. In order to edit the files on your host (your dev station) and to apply the code modifications to the app that is running in your Docker stack, the trick is to use the same UID (UserIDentification) on the local computer (the dev station) and the container, in order to preserve the rights on the filesystems on both side.
To do this, I added a line in the php Dockerfile to create a user **devuser** on the php container who owns the files from the **/var/www/html** folder. This user is a member of the **www-data** group on the container side.

In order to get your uid on the host machine by typing "id -u" on your host. Then replace the **1000** value by your own uid on this line :
```
RUN adduser --uid 1000 --disabled-password --home /home/devuser devuser
```

4. Start Docker containers

```bash
cd [location where you cloned the project]/laravel-apache2-pgsql
docker-compose build && docker-compose up -d
```

5. Create the the laravel project skeleton
(inside laravel-apache2-pgsql folder)
```bash
docker-compose exec -u devuser php composer create-project --prefer-dist laravel/laravel /var/www/html/.
```

6. Generate the key for the Laravel application
Next, set the application key for the Laravel application with this command :

```bash
docker-compose exec -u devuser php php artisan key:generate
```

This command will generate a key and copy it to your .env file, ensuring that your user sessions and encrypted data remain secure.

7. Sync de database.
Finally, to sync the database, you need to update the .env file. An exemple is shown below :

```
DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=pgdb
DB_USERNAME=pguser
DB_PASSWORD=pgpwd
```

Now, type the following URL. The port is the one we set up in the docker-compose.yml - If you check the docker-compose file, you can see in the apache service section that port 8080 of the host maps port 80 of the container.

http://localhost:8080

You can connect an external database client such as pgadmin or dbeaver.

8. Use Laravel *artisan* command to create a controller:

```bash
docker-compose exec -u devuser php php artisan make:controller SomeController
```

This command will create the *SomeController* file into the */var/www/html/app/Http/Controllers* container directory.

## Docker compose cheatsheet

**Note:** you need to cd first to where your docker-compose.yml file lives.

* Start containers in the background: `docker-compose up -d`
* Start containers on the foreground: `docker-compose up`. You will see a stream of logs for every container running.
* Stop containers: `docker-compose stop`
* Kill containers: `docker-compose kill`
* View container logs: `docker-compose logs`
* Execute command inside of container: `docker-compose exec SERVICE_NAME COMMAND` where `COMMAND` is whatever you want to run. Examples:

* Open a pgsql shell, `docker-compose exec -u postgres pgsql bash`
Then you can access the database with psql commad :
* `psql -U pguser -d pgdb -W`

## Docker general cheatsheet

**Note:** these are global commands and you can run them from anywhere.

* To clear containers: `docker rm -f $(docker ps -a -q)`
* To clear images: `docker rmi -f $(docker images -a -q)`
* To clear volumes: `docker volume rm $(docker volume ls -q)`
* To clear networks: `docker network rm $(docker network ls | tail -n+2 | awk '{if($2 !~ /bridge|none|host/){ print $1 }}')`