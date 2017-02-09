# New Projects

To set up a new project rather than just work on one, you'll need to understand how to create a 
docker-compose file, how to configure DNS for your project's containers and how to configure 
persistent storage if your project needs it.

## Manual Setup

Outrigger is intended to work with either raw Docker commands or Docker Compose files. Docker
Compose is the preferred tool to use when setting up projects and in this section we will go
over some basics to get a project setup.

Be sure to read the [Docker Compose compose file documentation](https://docs.docker.com/compose/compose-file/)
so that you have a handle on how to setup the various configuration.

Please see the [Outrigger Examples Repository](https://github.com/phase2/outrigger-examples) for 
how to setup various technologies within Outrigger.  Specifically, the 
[Drupal 8 example](https://github.com/phase2/outrigger-examples/tree/master/drupal8) covers all
of the high points.

### Choosing Images

Any valid Docker Image can work with Outrigger. We also supply [a collection of crafted Docker
images](../project-setup/docker-images.md) that are easily configured and optimized. And example for
an Apache / PHP image based on PHP7 is **phase2/apache-php:php70** or **php:7.1.1-apache**  

!!! important "Use Image Tags"
    When selecting and image to use, be sure to also specify a tag for that image. When you do not
    specify a tag, the default tag of *latest* is used and it is easy to get our of sync with other
    team members or simply get unwanted changes if you do not explicitly specify an image tag.

### Environment Variables

Images can take configuration through [environment variables](https://docs.docker.com/compose/compose-file/#/environment)
specified in the Docker Compose file. For example, the **phase2/apache-php** image allows you to configure settings such 
as `max_execution_time` with the `PHP_MAX_EXECUTION_TIME` environment variable and `memory_limit` with the 
`PHP_MEMORY_LIMIT` environment variable. You can also turn on settings like Xdebug with the `PHP_XDEBUG` environment 
variable.

### DNS

DNS Names for containers are managed with a service called dnsdock.  Dnsdock listens for containers started with certain 
label metadata specified and creates DNS entries for those containers. Read the linked docs to learn about 
[configuring label metadata in Compose](https://docs.docker.com/compose/compose-file/#/labels-1) and the key/values for 
[dnsdock label configuration](https://github.com/aacebedo/dnsdock#overrides-with-docker-labels)

### Volumes

Volume mounting directories and files into containers are use for a variety of reasons.

* Getting your code base into a container
* Specifying certain configuration files
* Overriding existing configuration files
* Making SSH keys available within a container
* Providing directories that will hold data that lives longer than the container (e.g. database files, or website uploads)

See the [Compose documentation for volumes](https://docs.docker.com/compose/compose-file/#/volumes-volumedriver) to learn
about the various options. Also check out the Outrigger examples linked above to see volume mounts in action.

## Using Generators

Coming Soon with details on Yo P2 and Generator Gadget as tools that prompt for a few details and then generate the 
scaffolding of an entire project configuration.