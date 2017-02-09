# Glossary

## Host Machine

Your laptop for the purposes of Outrigger. This is where your project's source code, your IDE, browsers, etc run as well 
as where the virtual machine acting as the Docker Host runs.

## Docker Host

This is a Linux-based virtual machine capable of running Docker. One Docker host VM can run multiple containers for 
multiple projects.

## Docker Image

A read only template that can be instantiated as a running container. An image might supply a service as small as a build 
tool or as large as a database and/or web server. Images are generally single purpose in nature and are then linked together 
to build more complex capabilities.

## Container

A runtime instance of a Docker Image. This is what contains a service like Apache or MySQL. Several containers can run on 
a single Docker Host and they can be linked so that they they know how to communicate with each other.

## Docker Machine

Docker Machine creates the Docker Host virtual machine in which containers run.  This provides the functionality that we 
previously used Vagrant to accomplish. Under the covers docker-machine starts up a virtual machine running a tiny version 
of Linux.  On Macs this tiny version of Linux is called boot2docker.  All of the Docker commands to build and run containers 
will actually be executed on the boot2docker virtual machine. On your laptop you will have a single boot2docker virtual 
machine (running in VirtualBox or VMWare Fusion) and it will host one or more Docker containers. Each project will likely 
have multiple Docker containers running (web server, database, cache, search, etc.).

## Docker Compose

Docker Compose is used to manage and coordinate the containers that need to run for a project in an easy to use YAML 
file. Each project will have a docker-compose file for each Docker environment that the project supports.  For example, 
the default docker-compose.yml would be used to start containers for local development, and alternate Docker compose 
files could be included for starting containers for the integration/stage and other environments.