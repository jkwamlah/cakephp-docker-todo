# coding-gems-todo
A docker setup for containerized CakePHP Applications

This setup generates the following containers

* **mysql** (8.0)
* **nginx**
* **php-fpm** (php 7.4)

The guide will walk you through the following things

* [Quick Start](#quick-start)
* [Installation](#installation)
* [How to use `bin/cake`, `mysql` and other commandline utils now that you're using containers](#now-how-to-run-bincake-and-mysql)

## Quick Start
1. Download the ZIP file for this repository
2. From your terminal/commandline, `cd` into the `docker` directory and run `docker-compose up`
3. Go to `localhost:8180` and your app will be live.


## Installation
Clone this repository or just download the .zip file and place it inside your app's root folder

From your terminal/commandline, cd into coding-gems-todo/docker and run docker-compose up

Here is an example of what my typical setup looks like

```
	myapp-folder
		cakephp
			src
			config
			..
		docker
			.env
			.env.sample
			docker-compose.yml
			mysql
				my.cnf
			nginx
				nginx.conf
			php-fpm
				Dockerfile
				php-ini-overrides.ini
```

Next, **Update the Environment File**

Copy or Rename `docker/.env.sample` to `docker/.env`.
This is an environment file that your docker-compose setup will look for automatically which gives us a great, simple way to store things like your mysql database credentials outside of the repository.

**Build and Run your Containers**

```bash
cd /path_to_your_app/docker
docker-compose up
```

That's it. You can now access your CakePHP app at

`localhost:8180`

> **tip 1**: start docker-compose with `-d` to run (or re-run changed containers) in the background.
> `docker-compose up -d`

> **tip 2**: Run `docker-compose build` to re-build containers. This is important when there are changes to your configuration files (.env or config/app_local or config/app)

**Connecting to your database**

Also by default the first time you run the app it will create a `MySQL` database with the credentials you specified in your `.env` file (see above)

``` yaml
host : myapp-mysql
username : myapp
password : myapp
database : myapp
```

You can access your MySQL database (with your favorite GUI app) on

`localhost:8106`

Your `cakephp/config/app_local.php` file should be set to the following (it connects through the docker link)

```php
  'Datasources' => [
    'default' => [
      'host' => 'myapp-mysql',
      'port' => '3306',
      'username' => 'myapp',
      'password' => 'myapp',
      'database' => 'myapp',
    ],
```

To change these defaults edit the variables in the `docker/.env` file or tweak the `docker-compose.yml` file under `myapp-mysql`'s `environment` section.

## Now, how to run `bin/cake` and `mysql`

Now that you're running stuff in containers you need to access the code a little differently

You can run things like `composer` on your host, but if you want to run `bin/cake` or use MySQL from commandline you just need to connect into the appropriate container first

**access your php server**

```bash
docker exec -it myapp-php-fpm /bin/bash
```
> remember to replace `myapp` with whatever you really named the container

**access mysql cli**

```bash
docker exec -it myapp-mysql /usr/bin/mysql -u root -p myapp
```
> remember to replace `myapp` with whatever you really named the container and with your actual database name and user login


## Troubleshooting

There are 3 containers that will be spooled up automatically

### `myapp-nginx` - the web server

First we're creating an nginx server. The configuration is set based on the CakePHP suggestions for nginx and `myapp-nginx` will handle all the incoming requests from the client and forward them to the `myapp-php-fpm` server which is what actually runs your PHP code.

You can configure the **nginx server** by editing the `/nginx/nginx.conf` file

### `myapp-php-fpm` - the PHP processor

This container runs `php` (and it's extensions) needed for your CakePHP app

It automatically includes the following extensions

* `php7.4-intl` (required for CakePHP 4.0+)
* `php7.4-mbstring`
* `php7.4-sqlite3` (required for DebugKit)
* `php7.4-mysql`

It also includes some php ini overrides (see `php-fpm\php-ini-overrides.ini`)

This container will (by default) look for your web app code in `../cakephp/` (relative to the `docker-compose` file).

You can configure what **PHP extensions** are loaded by editing `/php-fpm/Dockerfile`

You can configure **PHP overrides** by editing `/php-fpm/php-ini-overrides.ini`

### `myapp-mysql` - the database server

The first time you run the docker containers it will create a folder in your root structure called `mysql` (at the same level as your `docker` folder) and this is where it will store all your database data.

Since the data is stored on your host device you can bring the mysql container up and down or completely destroy and rebuild it without ever actually touching your data - it is "safely" stored on the host.

You can configure **MySQL overrides** by editing `/mysql/my.cnf`


## My CakePHP app cannot connect to the database
In your terminal/commandline, do the following
* `cd docker`
* `run docker ps -a`

Locate and copy the CONTAINER ID of the `myapp-mysql` container. (eg. 13abc689f252 | 55f6f0bb4a1e | d0fb131fb25d)
* `run docker inpsect [your_myapp-mysql_continer_id]`

Scroll down and copy the ip address of the container.
* ` "IPAddress": "172.25.0.3"`
* `cd ../cakephp/config`

Locate `app_local.php`. Under Datasources, change the host name to the IPAddress of the container you copied
```php
  'Datasources' => [
    'default' => [
      'host' => '172.25.0.3',
      'port' => '3306',
      'username' => 'myapp',
      'password' => 'myapp',
      'database' => 'myapp',
    ],
```
That's it!
```          
