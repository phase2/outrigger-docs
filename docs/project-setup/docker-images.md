# Docker Images

## Key components

The following Docker Images have been built to work with Outrigger. They all have a similar and consistent 
setup, so when using these images it is important to know the technology in place and how to customize it for you purposes.  
We have provided environmental configuration for the most frequently customized options, and any extended customization 
can be made by following the recommendations in [Customizing container configuration](../common-tasks/customizing-container-configuration)

These images are built with the following software

* [S6-overlay](https://github.com/just-containers/s6-overlay) Init System
* [confd](http://www.confd.io/) config file templating

!!! note
    Below are a collection of tuned images for working with Outrigger, but **any** Docker Image can be used within Outrigger.
    Additionally these Docker Images do not need to be used with Outrigger, they can be used in any Docker environment.

## Images

### phase2/servicebase 

([Docker Hub](https://hub.docker.com/r/phase2/servicebase/)) ([Repo](https://bitbucket.org/phase2tech/docker-servicebase)) 

A CentOS 7 base image that has s6-overlay and confd. This is a useful image for extending to build non-trivial services.

### phase2/servicebaselight 

([Docker Hub](https://hub.docker.com/r/phase2/servicebaselight/)) ([Repo](https://bitbucket.org/phase2tech/docker-servicebaselight))

This is an Alpine-based image that has had the S6-overlay init system and confd added to it.

In addition to the lightweight Alpine base it also includes bash and glibc so that Go-based Linux binaries will run. This 
image is only ~8MB when compared to the 100+MB of servicebase.

### phase2/apache-php 

([Docker Hub](https://hub.docker.com/r/phase2/apache-php/)) ([Repo](https://bitbucket.org/phase2tech/docker-apache-php))

Apache & PHP runtime images. It currently support PHP 5.5, 5.6, and 7.0

### phase2/apache-php-base 

([Docker Hub](https://hub.docker.com/r/phase2/apache-php-base/)) ([Repo](https://bitbucket.org/phase2tech/docker-apache-php-base))

A base image for phase2/apache-php. Includes Apache and a default VirtualHost configured with a proxy to PHP-FPM. It 
does not include the php runtime, that is handed in the extension image(s).

### phase2/build 

([Docker Hub](https://hub.docker.com/r/phase2/build/)) ([Repo](https://bitbucket.org/phase2tech/dockerrig-build))

This image provides the collection of development tools necessary to build applications, bundled with a wide array of 
tools useful for development and troubleshooting via the command-line interface. While it is possible to directly
connect via the "web" containers (apache-php), this is the preferred way to perform "server work".

It also contains some extras you may need to work with Drupal, including use of tools such as 
[Drupal Console](https://drupalconsole.com/), [Grunt Drupal Tasks](https://github.com/phase2/grunt-drupal-tasks) and 
[Pattern Lab Starter](https://github.com/phase2/pattern-lab-starter).

### phase2/mariadb 

([Docker Hub](https://hub.docker.com/r/phase2/mariadb/)) ([Repo](https://bitbucket.org/phase2tech/docker-mariadb))

MariaDB image for MySQL based builds with confd templates for config

### phase2/memcache 

([Docker Hub](https://hub.docker.com/r/phase2/memcache/)) ([Repo](https://bitbucket.org/phase2tech/docker-memcache))

Memcache image with configurable settings

### phase2/redis 

([Docker Hub](https://hub.docker.com/r/phase2/redis/)) ([Repo](https://bitbucket.org/phase2tech/docker-redis))

Redis image with a confd template for redis.conf

### phase2/node 

([Docker Hub](https://hub.docker.com/r/phase2/node/)) ([Repo](https://bitbucket.org/phase2tech/docker-node))

Node image 

### phase2/varnish 

([Docker Hub](https://hub.docker.com/r/phase2/varnish/)) ([Repo](https://bitbucket.org/phase2tech/docker-varnish))

Varnish container with fancy environment variables for easy configuration

### phase2/jenkins-docker 

([Docker Hub](https://hub.docker.com/r/phase2/jenkins-docker/)) ([Repo](https://bitbucket.org/phase2tech/docker-jenkins-docker))

Jenkins image that is built to be able to run Docker commands and launch containers. Docker-inception.
