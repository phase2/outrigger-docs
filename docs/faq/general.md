# General FAQ

## How do I configure my web container / virtual hosts so I can run multiple projects at once from the same web container

You don’t. Seriously. This is a place where you must change your thinking from that of treating a *server* as a unit to 
treating the **service** as a unit. If you have multiple projects active at once, you’ll have a web serving container 
active for each of them. This is OK because containers are much lighter weight than full virtual machines, though you do 
want to be careful about starting too many at once.  Consider that you will have a docker-compose file for each project 
you are working on, and each docker-compose file represents your application and all the services required to run it.  
You could be running multiple docker-compose applications on a single Docker Host.  Realistically, for the performance 
of your computer you would stop one docker-compose environment before you would run another.

## How do I see what containers are running on my Docker Host

If you want to see how many containers are present within your Docker Host VM and check on the status of them just 
run `docker ps` This will show you the containers running, the image they are based on, the ports they expose and the 
name of the container.

To see all the containers on the Docker Host both running and stopped use the command  `docker ps -a`

## Why do I have so many containers / cleanup

Any time you finish a project or you need to *reset* things, you should clean up your containers for a project by running 
`docker-compose rm`. That will remove the instances for the containers specified in your docker compose file.

Additionally you can run `rig prune` to clear out all stopped containers and any dangling images.

## Is there any regular maintenance needed

Yes, to keep your environment working smoothly and with current infrastructure configurations, implement a personal 
regimen of [Routine Image Maintenance](../common-tasks/routine-image-maintenance).

## Working with multiple Docker versions [Homebrew]

If you have multiple versions installed, doing `brew info docker-compose` will list the versions you have installed. The 
one with the **"*"** will be the version that is active.

If you need to switch versions, use the `brew switch` command and run any additional commands like `brew unlink ...`. 
(The `switch` command will provide next command steps if you need to link or unlink any formulae).

Example:

```bash
brew switch docker 1.13.1
```

## Can I get .vm container names a Docker Host not created with Outrigger

Yes, with an important caveat that containers will not be able to resolve .vm addresses without reconfiguring the daemon.

The `rig dns` command can accept a `--name` parameter to run the DNS services on an existing Docker Host

When we start a machine with `rig start`, we do the following things:

- Start a machine with the `-dns=172.17.0.1` Docker daemon option set so that all containers will try dnsdock for DNS
- Run dnsdock bound to `172.17.0.1:53` to provide .vm addresses for all containers
- Set up Mac OS X to route 172.17.0.1/16 to the Docker virtual machine's IP so the host machine can access containers direct
- Set up `/etc/resolver/vm` so that OS X will look up .vm addresses through DNS queries to `172.17.0.1`

If you run `rig --name=$DOCKER_MACHINE_NAME dns`, every step except the first will run, so your OS X host will be 
able to reach containers by their .vm addresses but other containers will not. This is the major difference between 
creating a machine with `rig start` and applying the DNS configuration to an existing machine with `rig dns`.

## Monitoring of containers

If you need some insight into how many resources a given Docker container may be using, take advantage of the command 
`docker stats <container name>`.  This handy command will show you CPU%, Memory%, Memory Usage vs Limit, and Network I/O.  
It is rudimentary but can be very useful in the first line of inspection on a container.
