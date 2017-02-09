# Starting your project containers

## Configure your environment

Ensure all terminals you intend to use can communicate with the Docker Host.  Generally a simple `docker ps` will either
list out running containers or give you an error like *Cannot connect to the Docker daemon. Is the docker daemon running 
on this host?*

If you get the error make sure you have run `eval "$(rig config)"`. See 
[Installation](../getting-started/mac-installation.md) for the proper way to configure your environments.

## Start your containers

In the project directory, start the containers with: `docker-compose up`

!!! note
    The docker-compose command runs in the foreground as long as the Docker containers are running. **(It is not hung 
    if there is no output after "Attaching [container name]...)**

Logs from the running containers will stream to the console and be prefixed by their compose names plus an 
integer (i.e. web_1, db_1, etc)

You can start another terminal tab if you need to run other commands (such as the build container operations, etc.)
