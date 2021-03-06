# Part II: Drupal Docker Containers

<!-- toc -->

During [Part I](./local_docker_development.md#part-i-shared-docker-containers) we just started the shared Docker containers. For each Drupal Site we need an own Docker Container:

## Prerequisites
* [Docker Compose](https://docs.docker.com/compose/install/) (Version >= 1.7.0)
  * On OS X just run `brew install docker-compose`
  * On Linux use your favorite package manager or https://docs.docker.com/compose/install/
  * On Windows it's already installed with "Docker for Windows"

## Find the right `docker-compose.yml`

1. Visit https://github.com/amazeeio/docker or clone https://github.com/amazeeio/docker.git into a folder on your computer
2. Copy the desired example file into your Drupal directory (see descriptions below). Pick a 'basic' file if unsure.
3. Rename the file to `docker-compose.yml`
4. Edit the file according to your needs, change at least the hostname. _Btw: It's perfectly fine to commit this file into your git repo, so others that are also using amazee.io docker can use it as well._
5. Run in the same directory as the `docker-compose.yml`:

        docker-compose up -d
6. If you are on Windows add the URL to the Hosts file (see [windows documentation](local_docker_development/windows.md) for that).
7. Open your browser with the entered URL in the `docker-compose.yml`, happy Drupaling!

## Connect to the container

To run commands like `git` or other things within the container, you need to connect to the container.

There are two ways for that:

### Connect via `docker-compose` (easier)

This is the easier way, you need to be in the same folder where also the `docker-compose.yml` for that to work:

    docker-compose exec --user drupal drupal bash

### Connect via `docker`

If you want to connect to a container wherever you are right now with your bash:

	docker exec -itu drupal changeme.com.docker.amazee.io bash

*Replace `changeme.com.docker.amazee.io` with the docker container you want to connect to*

### Drush from your host machine

To use drush, you can either connect to the container as above, or add a bash function that will connect for you to run your drush command. To add the function, add this to your .bashrc file:

Bash:
```
function ddrush() {
  args=""
  while [ "$1" != "" ]; do
    args="${args} '$1'" && shift
  done;

  docker-compose exec --user drupal drupal bash -c "source ~/.bash_envvars && cd /var/www/drupal/public_html/\"\$WEBROOT\" && PATH=`pwd`/../vendor/bin:$PATH && drush ${args}"
}
```

Fish Shell - ([fishshell.com](https://fishshell.com/)):
```
function ddrush --description 'Drush fish (friendly interactive shell) function that detects Amazee.io Docker container. '
  if test -f (git root)/.amazeeio.yml
    echo "Using Amazee.io Docker Container Drush"
    command docker-compose exec --user drupal drupal bash -c "source ~/.bash_envvars && cd /var/www/drupal/public_html/docroot && PATH=`pwd`/../vendor/bin:\$PATH && drush $argv"
  else
    command drush $argv
  end
end

funcsave drush
```


When you next start a bash session, you'll be able to use `ddrush` just like your normal `drush` command.

## SSH Agent

Per default your SSH Key at `~/.ssh/id_rsa` is added to the Docker containers from either `cachalot` or `pygmy`

If you need another key, read the documentation of [`cachalot`](os_x_cachalot.md) or [`pygmy`](linux_pygmy.md) about this.

## Update Images

We constantly make improvements, updates and some other nice things to our container images. Visit [changelog.amazee.io](https://changelog.amazee.io) to see if there is something new. If you need to update the Docker Images to the newest version from the Docker Hub run in the same folder as the `docker-compose.yml`:

	docker-compose pull
	docker-compose up -d

### Slow Updates?

When pulling a new docker image, the download can get stuck. There is a nice workaround for that: pull a second time :)

Just open another terminal window at the exact same directory than you run the first `docker-compose pull` and just run that command again. The download will be unstuck and continue again. If the download is stuck again, cancel the second command with CTRL+c, and run it again (no worries, the first one will continue to run). Repeat that until the download is completely done.


## `docker-compose.yml` example files

| Example File                                                                                  | PHP | Services                      | Description                                                                                                                     |
|-----------------------------------------------------------------------------------------------|-----|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| [`php70-basic`](https://github.com/amazeeio/docker/blob/master/example-php70-basic.yml)       | 7.0 | nginx, varnish, mariadb       | prefered for Drupal 8                                                                                                           |
| [`php56-basic`](https://github.com/amazeeio/docker/blob/master/example-php56-basic.yml)       | 5.6 | nginx, varnish, mariadb       | prefered for Drupal 7                                                                                                           |
| [`php70-composer`](https://github.com/amazeeio/docker/blob/master/example-php70-composer.yml) | 7.0 | nginx, varnish, mariadb       | For [Drupal Composer Project](https://github.com/drupal-composer/drupal-project), the Drupal root expected in the folder `/web`, or another folder (configurable in the `docker-compose.yml`) |
| [`php56-composer`](https://github.com/amazeeio/docker/blob/master/example-php56-composer.yml) | 5.6 | nginx, varnish, mariadb       | For [Drupal Composer Project](https://github.com/drupal-composer/drupal-project), the Drupal root expected in the folder `/web`, or another folder (configurable in the `docker-compose.yml`) |
| [`php70-solr`](https://github.com/amazeeio/docker/blob/master/example-php70-solr.yml)         | 7.0 | nginx, varnish, mariadb, solr | The same as php70-basic but additionally Apache Solr installed. Check the yaml file on how to define your solr configs          |
| [`php56-solr`](https://github.com/amazeeio/docker/blob/master/example-php56-solr.yml)         | 5.6 | nginx, varnish, mariadb, solr | The same as php56-basic but additionally Apache Solr installed. Check the yaml file on how to define your solr configs          |
