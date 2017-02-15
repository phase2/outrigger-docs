# SSH into a Container

## How do I SSH into a running container

There is a docker exec command that can be used to connect to a container that is already running.  

* Use `docker ps` to get the name of the existing container
* Use the command `docker exec -it <container name> /bin/bash` to get a bash shell in the container
* Generically, use `docker exec -it <container name> <command>` to execute whatever command you specify in the container.

## How do I run a command in my container

The proper way to run a command in a container is: `docker-compose run <container name> <command>`. For example, to get 
a shell into your web container you might run `docker-compose run web /bin/bash`

To run a series of commands, you must wrap them in a single command using a shell. For example: 

```bash
docker-compose run [name in yml] sh -c '[command 1] && [command 2] && [command 3]'
```

In some cases you may want to run a container that is not defined by a docker-compose.yml file, for example to test a new 
container configuration. Use docker run to start a new container with a given image: `docker run -it <image name> <command>`

The docker run command accepts command line options to specify volume mounts, environment variables, the working 
directory, and more.

## Getting a shell for build/tooling operations

Getting a shell into a build container to execute any operations is the simplest approach. You simply want to get access 
to the `cli` container we defined in the compose file.  The command `docker-compose -f build.yml run cli` will start an 
instance of the `outrigger/build` image and run a bash shell for you.  From there you are free to use `drush`, 
`grunt` or whatever your little heart desires.

## Running commands, but not from a dedicated shell

Another concept in the Docker world is starting a container to run a single command and allowing the container stop when 
the command is completed.  This is great if you run commands infrequently, or don't want to have another container 
constantly running.  Running your commands on containers in this fashion is also well suited for commands that don't 
generate any files on the filesystem or if they do, they write those files on to volumes mounted into the container.

The `drush` container defined in the example `build.yml` file is a container designed specifically to run Drush in a 
single working directory taking only the commands as arguments.  This approach allows us to provide a quick and easy 
mechanism for running any Drush command, such as `sqlc`, `cache-rebuild`, and others, in your Drupal site quick and easily.

There are also other examples of a `grunt` command container similar to `drush` and an even more specific command 
container around running a single command, `drush make` to build the site from a make/dependency file.
