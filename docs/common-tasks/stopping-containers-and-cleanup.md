# Stopping containers and cleanup

## Stopping the containers for your project

You may want to recoup the resources used by your projects containers and the docker host for other things if you are 
done with development for a while.  You can stop the containers for a single project only or all containers and the docker 
host depending on what you are finished using.

1. If you only want to stop the containers for your project, in the project directory, run `docker-compose stop` or 
press Ctrl-C to stop a docker-compose process running in the foreground and then run `docker-compose stop` to ensure 
the project containers have stopped.
1. If you want to shut down the docker host as well as any containers, run `rig stop`

## Cleaning Up

From time to time you'll want to clean up stopped containers. You'll also want to take special care when finishing a 
project to release all the resources used by it.

For periodic cleanup of all stopped containers, run the following script while your docker host is running: `rig prune`

If you only want to clean up project specific stopped containers, you can run: `docker-compose rm` from your project directory.

When you are finished with a project, if you used any persistent data storage you'll want to run a command to clean it 
up. The exact directory to request removal from will depend on your project (see suggested directory naming guidelines in 
[Key Concepts](../project-setup/key-concepts.md#persistent-data-volume)): 

```
docker-machine ssh dev
sudo rm -rf /data/[project]/
```
