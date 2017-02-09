# Using the Build Container

Part of Outrigger is a [build](https://hub.docker.com/r/phase2/build/) image.  The idea of the build 
container is that it will have installed many of the tools you'd need to work on a project and the proper versions 
so that they work well together.  Everything from drush to gem to npm to composer as well as grunt, bower, yeoman, etc. 
Providing these tools in a container means that you'll never have to worry about having the right tools installed on 
your laptop or integration environment to get your job done.

The example Drupal project has a [build.yml](https://github.com/phase2/outrigger-examples/blob/master/drupal8/build.yml)
file that shows a variety of ways to use the container.  

## Getting a build container command line

One of the most common ways to use the build container is to launch a shell into the container and run commands from the 
command line interface (cli). The following command get you to the cli

`docker-compose -f build.yml run cli`

!!! note "Making a command alias"
    Using a shell alias you can make a really quick CLI command that is reusable across projects if you name your build.yml
    file the same everywhere. `alias cli='docker-compose -f build.yml run cli` then you only need to `cli` to get into
    your build containers shell.  This *trick* can be used with all of the command below as well.

## Running a command container

A "command container" is a term we have coined for a specific approach we use, where containers are spun up to run a single 
command in a consistent environment and then shut down.  Here are some example of command containers that we have configured 
into the `build.yml` file from our example repository above.

### Drush Command Container

This is a general purpose Drush command container that allows you to run any Drush command in a consistent environment

`docker-compose -f build.yml run drush [command here]`

### Grunt Command Container

This is a general purpose Grunt command container that allows you to run any grunt command in a consistent environment

`docker-compose -f build.yml run grunt [command here]`

### Composer Command Container

This is a general purpose Composer command container that allows you to run any composer command in a consistent environment

`docker-compose -f build.yml run composer [command here]`

### Composer Install Command Container

This one goes a step further when there is a specific command you want to run and control all the arguments used to run it.

`docker-compose -f build.yml run composer-install`

## Share output from the build container

Using the example project from above, we have a volume mount of your project code into the build container. Now, when 
you perform a build and it writes the output to the local project directory (e.g in ./build/html or something else in 
the working directory), those changes to the filesystem are effectively outside of the container. The web container should 
also define a volume mount for the build output into the docroot, or mount your project directory in the container and 
configure the docroot to be the absolute path to your build output and you will have seamless integration of build output 
to a web container running your project.
