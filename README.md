# Drupal Site Template

Drupal Website

## Initialising your project

1. Copy the files of this repository
2. Customise the project information in `composer.json`
3. Run `composer update` in order to create your first composer.lock
4. You can add new dependencies at this stage before continuing if you already
   know you will use them
5. Remove this section of the file

## Website Development

We assume the following development style (using the docker terminology to refer
to your computer as the "host" to distinguish it from the containers):

- Files are edited on your host system using your editor of choice.
- **All** commands (`composer`, `npm`, `drush`, `phpunit` etc.) are run from
  within their respective container via `docker-compose exec`.

This template has a separate (single command) step to install Drupal which you
must run after you have started the containers (see below).

We did this to avoid baking things like DB credentials into the images. If you
decide you do want to bake some of those steps into the image then you can move
commands from `docker/config/services/drupal/root/setup.sh` into
`Dockerfile.drupal.dev`.

The `docker/config/` dir is organised first by service (see `docker-compose.yml`
for the list of services) and then by the path within the container that the
file/folder will be copied/mounted i.e.

```
docker/config/services/{NAME_OF_DOCKER_COMPOSE_SERVICE}/{PATH_THAT_FILE_IS_COPIED_OR_MOUNTED_IN_THE_SERVICE}
```

### Getting started

Please note that each command below is prefixed by a shell prompt which tells
you whether this command should be run on your docker host (`your-computer`
below) or one of the containers.

```sh
## Terminal 1 ########

## start all the services (add --detach arg to this command if you
## want it to return control of your terminal to you)
you@your-computer$ docker-compose up --build

## Terminal 2 ########

## install Drupal (edit docker/config/services/drupal/root/setup.sh if you want to change
## how Drupal is installed e.g. download and enable modules etc.)
you@your-computer$ docker-compose exec drupal /root/setup.sh

## Now your drupal site should be available.

## To run commands e.g. composer or mysql, we open a shell in the drupal
## container and run them from there
you@your-computer$ docker-compose exec drupal bash

## When you have a shell in the drupal container you can ...
## ... follow along with the nginx logs
drupal-container$ tail -f /var/log/nginx/error.log
drupal-container$ tail -f /var/log/nginx/access.log

## ... connect to mysql
drupal-container$ mysql

## ... working with JS
drupal-container$ cd themes/custom/site-theme
drupal-container$ npm run sass # or whatever commands you have defined in npm
```

If you prefer you can run commands directly via docker-compose without having to
open the bash shell first e.g.

```sh
## Alternatively you can pass the command you want to run directly to
## `docker-compose exec` e.g. run mysql CLI client
you@your-computer$ docker-compose exec drupal mysql
```

After you install Drupal (see above), the following should be available in your
web browser:

- http://localhost:8080/ (Drupal in the `drupal` container)
- http://localhost:3001/ (Browsersync in the `frontend` container)

### Adding Zurb Foundation

If you want to add the theme Zurb Foundation Theme run this. In order to

```
docker-compose exec drupal composer require "drupal/zurb_foundation:^6.0"
docker-compose exec drupal drush en zurb_foundation -y
```

#### Create a child theme from Zurb's foundation

Based on
[these instruction](https://www.drupal.org/docs/8/themes/zurb-foundation-user-guide/zurb-foundation-8x-6x/creating-a-custom-foundation-subtheme#s-create-a-subtheme-using-drush)
, you can run these commands to create a child theme.

Replace `my_custom_theme` with the desired machine name for your theme:

```
docker-compose exec drupal drush fst my_custom_theme -n
docker-compose exec drupal mv themes/contrib/my_custom_theme themes/custom/my_custom_theme
docker-compose exec drupal drush en my_custom_theme -y
```

In order to avoid confusion, you can remove this section from this README.md
once you have created your theme.

### Adding custom module code

Create custom modules in `modules/custom` and they will automatically be
available to the Drupal container.

If your custom module is in a separate git repo then you can clone into that dir
and that will also be fine.

### Running Tests Manually

```sh
you@your-computer$ docker-compose exec drupal bash

## NB: VERY IMPORTANT: YOU MUST RUN TESTS AS www-data USER
## We have to run PHPUnit as a non-root user (otherwise it seems to fall back to
## running as the 'nobody' user who has no permissions to do anything)
root@drupal-container> su www-data
www-data@drupal-container> cd /var/www/html
www-data@drupal-container> phpunit --verbose modules/my-module-under-development/
```

### Running tests as CI does

This runs all tests the same way CI does.

```sh
you@your-computer$ docker-compose  exec drupal /root/run-ci.sh
```

### Drupal admin credentials

The default credentials (set by env vars in `docker-compose.yml`) are:

    Username: admin
    Password: admin

### Documentation

- Drupal API Reference https://api.drupal.org/api/drupal
- Drupal coding standards
  https://www.drupal.org/docs/develop/standards/coding-standards
- Drupal Core Changelog https://www.drupal.org/list-changes/drupal

### Caveats

- The Drupal site is in cache disabled mode for to make templating/frontend
  easier.
- **Accessing the site as an anonymous user still makes use of caching even when
  local development settings have been enabled. You must be logged in to view
  your site with caches disabled.**

### Cleaning up

```
## clean up (-v also deletes the volume that stores the MySQL data so only use
## it if that's the result you want)
you@your-computer$ docker-compose down -v
```
