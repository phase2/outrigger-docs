# Customizing Configuration

There are a few primary mechanisms for customizing the configuration of a container.

## Environment Variables

Many images will react to environmental variables to alter their behavior. Images which offer
this functionality should document it in the README for the image. For an example see the
[Outrigger Build](https://hub.docker.com/r/phase2/build/) image. When environmental
configuration is available it can be triggered by specifying the appropriate value in the
environments section of your docker compose file.

```yaml
www:
  image: phase2/apache-php:php56
  environment:
    # Change some core container settings
    PHP_MAX_EXECUTION_TIME: 45
    # These enable debug/profiling support
    PHP_XDEBUG: "true"
    PHP_XHPROF: "false"
```

## Volume Mounts

If the container image you are using doesn't allow for change via environmental variables, your 
next option is to override the configuration file using volume mounting. In this setup, you
replace the configuration file inside a container with a file from your host machine.

```yaml
www:
  image: phase2/apache-php:php56
  volumes:
    # substitute in a special mime magic file because our project handles files
    # of special types
    - ./env/www/etc/httpd/conf/magic:/etc/httpd/conf/magic
```

## Volume Mounts for confd

Phase2 images often use [confd](http://www.confd.io/) to template configuration from environment variable and other sources.
The configuration file you may be trying to override might be generated on container initiation via confd. If this is the 
case, you'll need to override the template file from which the configuration file is created. If you find that your copy 
of the configuration file on your host machine is updated when you start a container this is likely the cause. 

For example, the Xdebug configuration provided in our **phase2/apache-php:php70** container is provided by confd, so this
is how you would override that configuration

```yaml
www:
  image: phase2/apache-php:php70
  volumes:
    # substitute a different confd template file into the image so confd will use the override on container boot
    - ./env/www/etc/confd/templates/xdebug.ini.tmpl:/etc/confd/templates/xdebug.ini.tmpl
```
