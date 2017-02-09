# Architecture

Outrigger uses a containerization based approach to providing a project environment. Project services running as containers
are combined with configuration for persistent data storage, DNS services and network manipulation which allows for easy
access via names like service.project.vm rather than IP addresses.

The Outrigger CLI facilitates setting up a VM to act as a container host on systems which can not host containers natively.
It also configures the VM with the following:

* Filesystem sharing for high performance file access of the host machine
* Container initiation for DNS services
* Network and name resolution configuration

Layered with this core are a set of docker images which provide common project services such as a web server along with
the ability to configure these services with commonly desired development options. Any docker image can be used if desired.

Docker Compose is used to control containers to provide a complete environment and a set of Yeoman based generators are
used to help initial project setup to ensure all pieces fit together using the current best known options..

## Containerization

In order to provide lightweight environments and tooling we use a technical approach called containers.
Containers attempt to move the unit of environment from server to application. This allows separation of concerns between
how an application is configured, how the containers communicate with each other, and where the containers are deployed.
Take an advanced Drupal stack that includes Apache/PHP, MySQL, Memcache, and SOLR. With each component configured to run
in its own container, the containers can all run on a single VM for local development and be spread across multiple
servers for an optimized production deployment. Let us briefly touch on the technology we will be using and how it
conceptually fits together.

### Docker

The container implementation we use is called Docker which is explained in the intro: [What is Docker](https://www.docker.com/whatisdocker)?
This is the way we capture environment units for our application/services and share them with everyone on the team.
Environments are captured as images, similar to a VM, so when anyone runs that image they all start with the exact same
set of files. For example, nearly every project needs a web server so we have a container image that can be run
to provide that service. Our Docker images are also set up to allow for configuration adjustment to enable common
development options.

### rig

This is a project that glues all of the hosting aspects of these tools together into an easy to use unit. You can find
rig in a GitHub repository [here](https://github.com/phase2/rig). There are 2 basic services that rig provides:

#### Manage virtual machines for running containers

The rig binary will manage the creation/configuration/upgrade/start/stop of boot2docker virtual machines (a.k.a
Docker Hosts) via docker-machine. It ensures that the docker-machine virtual machine is the right version, is named
correctly and configured to run efficiently within VirtualBox, VMWare Fusion or xhyve.

#### Nice DNS names and routing for running containers

Once there is a safe environment to run our containers we need a way to route traffic to them and provide easy to
use/remember domain names to make accessing these services simple. Domain names for containers are set in the
docker-compose YAML files using configuration that the **dnsdock** container reads to create a mapping between domain
name and container.

We use **dnsdock** running as a container within the Docker Host. The **dnsdock** service, which listens on
172.17.0.1:53535. The **dnsdock** container resolves the \*\.vm domain names to the IP addresses of the containers.

Internal container names will look like *web.openatrium.vm*. All Outrigger containers will carry the **.vm** extension
for name resolution. There is additional information in [DNS Resolution](../common-tasks/dns-resolution.md).

### Docker Hub

Docker Hub is where container images are stored and retrieved when your local machine does not already have a copy of
the requested container image. Docker Hub can be thought of like GitHub or BitBucket, and Docker Hub images can be thought
of as git repositories. We can make new versions of the images and they can be pushed and pulled to the Docker Hub service.

### Container Configuration

If a container wants to offer configurable options it will document how to control it within the README or via Environment
variables in the Dockerfile itself. See our Apache / PHP [Dockerfile](https://hub.docker.com/r/phase2/apache-php/~/dockerfile/)
for an example. In this Docker Image, passing environment variables can override the PHP memory limit. Those variables
can either be passed on the command line when executing a `docker run` command directly, or in the `environment` section
of a docker-compose file. Documentation entries within the **Common Tasks** section provide additional approaches to
container configuration.
