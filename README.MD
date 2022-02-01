# Phalcon Nest
This package helps to launch and debug Phalcon [**LAMP**](https://en.wikipedia.org/wiki/LAMP_(software_bundle)) project under different versions

**Preconfigured Services:**
- MySQL 8
- Apache2 + Phalcon3 + php7.3 + DevTools
- Apache2 + Phalcon4 + php7.4 + DevTools
- Apache2 + Phalcon5 + php8.0

Each service configured with support of following libraries:
- [Composer](https://getcomposer.org/)
- [NewRelic](https://newrelic.com/)
- [xDebug](https://xdebug.org/)
- JSON, [DOM](https://www.php.net/manual/en/book.dom.php), [Tidy](https://www.php.net/manual/en/book.tidy.php), [Yaml](https://www.php.net/manual/en/book.yaml.php)
- Caching: [APCu](https://www.php.net/manual/en/book.apcu.php), Redis
- Database: [PDO](https://www.php.net/manual/en/book.pdo.php) with MySQL and SQLite support
- Image Processing: [ImageMagick](https://www.php.net/manual/en/book.imagick.php), [GD](https://www.php.net/manual/en/book.image.php), Exif
- [BCMath](https://www.php.net/manual/en/intro.bc.php "Arbitrary Precision Mathematics") and [GMP](https://www.php.net/manual/en/book.gmp.php "GNU Multiple Precision")
- [Curl](https://www.php.net/manual/en/book.curl.php)
- i18n support: [mbstring](https://www.php.net/manual/en/book.mbstring.php), [gettext](https://www.php.net/manual/en/book.gettext.php)
- [Zip](https://www.php.net/manual/en/book.zip.php)

## Setting up your project

### Option 1 
Create a new git project
```shell
git clone https://github.com/nazariy/phalcon-nest.git
```
### Option 2 
Add to existing project under `/docker/` directory
```shell
 git submodule add https://github.com/nazariy/phalcon-nest.git docker
```

## Configuration
Update your `.env` file where `docker-compose.yml` is located
```dotenv
# Docker defaults
COMPOSE_PROJECT_NAME=nest
COMPOSE_PROJECT_WORKING_DIR=/home
# MySQL
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=nest
MYSQL_USER=phalcon
MYSQL_PASSWORD=phalcon
MYSQL_HOSTNAME=172.50.50.8
MYSQL_PORT=3306
# Apache defaults
APACHE_DOCUMENT_ROOT=/home
APACHE_PUBLIC_DOCUMENT_ROOT=/home/public
```
> Note: mysql host is not accessible via `localhost` for that reason we are using IP address from our subnet
### NewRelic
This step is optional, but it's always good to have some insights on your application performance and issues

> Our PHP agent auto-instruments your code so you can start monitoring applications. 
> You can use our guided installation for an automated install, or follow the instructions in this document to complete a basic PHP agent installation. 
> Either way, you need a New Relic account. (It's free, forever.)

Update your `php.ini` and set your license key which can be obtained for [free](https://docs.newrelic.com/docs/apm/agents/php-agent/installation/php-agent-installation-overview/)
```ini
[newrelic]
newrelic.license = "YOUR_SHINY_LICENSE"
newrelic.logfile = "/var/log/newrelic/php_agent.log"
newrelic.appname = "Phalcon Application"
newrelic.daemon.logfile = "/var/log/newrelic/newrelic-daemon.log"
newrelic.daemon.app_connect_timeout = 15s
newrelic.daemon.start_timeout = 5s
```
### xDebug
Here is a minimum configuration for PHPStorm, add this lines to your `php.ini` file, accessed via port `9003`
```ini
[xdebug]
xdebug.mode                 = debug,develop
xdebug.idekey               = "PHPSTORM"
xdebug.start_with_request   = yes
xdebug.client_host          = host.docker.internal
xdebug.cli_color            = 1
```

## Starting container

> By default, when accessing `https://localhost` you would see a test page with `phpinfo()` output

### Phalcon3 (v3.4.5)
```shell
docker compose -f docker-compose.yml up -d phalcon3
```
### Phalcon4 (v4.1.3)
```shell
docker compose -f docker-compose.yml up -d phalcon4
```
### Phalcon5 (v5.0.0beta2) 
```shell
docker compose -f docker-compose.yml up -d phalcon5
```
> Please note that DevTools are not compatible with the latest version of Phalcon5

## Troubleshooting
Time to time things can go wrong depending on your current operating system and other factors.

Here are few issues I came across while playing with containers
#### failed to solve rpc error code
> example error code:
> ```log
> failed to solve rpc error code = unknown desc = failed to solve with frontend dockerfile.v0
> ```
> solution: Disable `buildkit` 
> 
> [stackoverflow](https://stackoverflow.com/questions/64221861/an-error-failed-to-solve-with-frontend-dockerfile-v0)