# Working with Volumes

Volumes are a way that you can map directories or individual files into a running container.  This is useful to provide 
code to run for a generic container, or directories to store data that persist longer than the life of the container, or 
to even override directories and/or files that exist in an image.  Lets look at a few of those examples.

## Provide code for a generic container to run*

The default phase2/apache-php:php70 is a Web/PHP container that provides only an index file in `/var/www/html` that 
prints out phpinfo().  This is obviously not very useful for an application, so we can provide an entire Drupal site to 
run and we do that by mapping our site into the default docroot like this:

`./build/html:/var/www/html`

This takes the Drupal site we have in our local project directory of `build/html` and it overrides the default content 
of the images `/var/www/html` directory.

## Persist data longer than the life of the container

When using a database like MySQL the container will store the database files in `/var/lib/mysql` and by default that 
directory will be reset every time the container restarts. This means each time you restart your container you'd need 
to reinstall your application and database, which can make life difficult. So in order to persist MySQL data for longer 
than the current run of the container we will map a directory from the Docker Host into the container and override the 
default `/var/lib/mysql` directory. The configuration will look something like this.

`/data/drupal/mysql:/var/lib/mysql`

Now when your database container creates files in `/var/lib/mysql` they are actually saved in the `/data/drupal/mysql` 
directory on the Docker Host. Be sure to namespace your directories inside /data so that separate projects don't conflict 
with each other. You'll also want to ensure you clean up when you are done with your project to keep from using up all 
of your disk space. See the cleaning up section for more info.

## Override directories and files that exist in an image

Generally the configuration shipped with a Docker Image is meant to be the production configuration. Often, that 
configuration is not suitable for development and we need to override configurations.  With volumes we have shown 
earlier how you can override directories, but you can also override individual files too.

`./config/dev/httpd/httpd.conf:/etc/httpd/httpd.conf`

This takes a local `httpd.conf` file from our project and overrides the `/etc/httpd/httpd.conf` files that ships with 
the container. 

## Changing Volume Definitions in Compose File

If you wind up changing volume definitions in your docker-compose file you will need to remove your container before it 
will recognize those changes on a restart.

In the project directory, run: `docker-compose rm`

This command will remove all stopped containers defined in the docker-compose.yml.  You could also remove an individual 
container by running: `docker-compose rm <name in yml>`

After removing the container, it will revert to the original state from the Docker image, removing any packages 
installed or files modified that are not included in a volume mount. Start your containers again with `docker-compose up` 
and you should have your new volume mounts.
