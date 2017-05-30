# Existing Projects

Outrigger is intended to work with either raw Docker commands or Docker Compose files. Docker
Compose is the preferred tool to use when setting up projects and in this section we will go
over some basics to get a project setup.

Be sure to read the [Docker Compose compose file documentation](https://docs.docker.com/compose/compose-file/)
so that you have a handle on how to setup the various configuration.

Please see the [Outrigger Examples Repository](https://github.com/phase2/outrigger-examples) for 
how to setup various technologies within Outrigger.  Specifically, the 
[Drupal 8 example](https://github.com/phase2/outrigger-examples/tree/master/drupal8) covers all
of the high points.

## Choosing Images

Any valid Docker Image can work with Outrigger. We also supply [a collection of crafted Docker
images](../project-setup/docker-images.md) that are easily configured and optimized. And example for
an Apache / PHP image based on PHP7 is **phase2/apache-php:php70** or **php:7.1.1-apache**  

!!! important "Use Image Tags"
    When selecting and image to use, be sure to also specify a tag for that image. When you do not
    specify a tag, the default tag of *latest* is used and it is easy to get our of sync with other
    team members or simply get unwanted changes if you do not explicitly specify an image tag.
    
## Common Config

TODO: Link to Env Vars
TODO: Link to Labels
TODO: Link to Volume Mounts
TODO: Link to Project Configuration
TODO: Copy Manual Setup (from New Project) here