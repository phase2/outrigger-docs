# Troubleshooting

See the following sections for common problems and ways to solve them.

## Run doctor

Run `rig doctor` to determine if your environment is set to run Outrigger.

## Ensure the environment is setup correctly

It can be useful to ensure everything is in a clean state. The following should ensure that

* `rig stop` stops the docker machine and cleans up networking
* `eval "$(rig config)"` clears environmental variables docker uses to communicate with the docker host
* `rig start` starts the docker machine
* `eval "$(rig config)"` sets environmental variables docker uses to communicate with the docker host

## Ensure your images are up to date

From time to time images are updated to fix bugs or add functionality. You won't automatically receive these updates but
you can fetch them when you hear new ones are available.

`docker pull imagename` can be used if you want to update a specific image. For example, if you wanted to make sure you
had the latest mariadb you'd run `docker pull phase2/mariadb`.

`docker-compose pull` can be used within a project directory to make sure you've got the latest version of all images
in the docker-compose.yml file.

## Configure Your Shell

If you do not have any containers listed when running `docker ps` or you get an error message like:

    Get http:///var/run/docker.sock/v1.20/containers/json: dial unix /var/run/docker.sock: no such file or directory.

    * Are you trying to connect to a TLS-enabled daemon without TLS?

    * Is your docker daemon up and running?

Or an error message like:

    Couldn't connect to Docker daemon - you might need to run `boot2docker up`.

Make sure your shell has the necessary environment variables by running:

`eval "$(rig config)"`

## Reset everything

If a problem continue to persists and the Docker Host is non-responsive, you may need to resort to the nuclear option of
blowing everything away and starting over. Note this is a **nuclear option** as even your persistent data area will be
removed if you don't back it up. If you have any data that needs to be maintained be sure to get a copy of it off of your
VM first with the scripts provided.

To wipe everything out and start over

1. First backup your existing data (if desired) by running `rig data-backup`. This will sync your entire `/data`
directory to your host machine.
1. Then you can run `rig remove` This removes the broken Docker Host and it’s state, making way for a clean start.
1. Next you can rebuild everything by running `rig start`. This will create you new Docker Host.
1. If you wish to restore the `/data` directory that you previously backed up, then run `rig data-restore`

## Files Not Found

If you get messages about files not being found in the container that should be shared from your host computer, check the
following:

1. Is the project checked out under the `/Users` folder? Only files under the `/Users` folder are shared into the
Docker Host (and thus containers) by default.
1. Does the `docker-compose.yml` file have a volume mount set up that contains the missing files?
1. Get shell in the Docker container via `docker exec -it <container name> /bin/bash` and check if the files are
shared in the wrong place.
1. Get shell in the Docker Machine with `docker-machine ssh dev` and check if you find the files under /Users.

## Image not found

If you encounter the following error about the Docker image not being found when starting a project, it may indicate
that the image is private and your Docker client is not logged into a required private Docker Hub repository:

    Pulling repository phase2/privateimage

    Error: image phase2/privateimage:latest not found

To solve this, run `docker login` and provide the relevant credentials.

## Docker client and server version incompatibilities

If you see an error similar to this:

    Error response from daemon: client is newer than server (client API version: 1.20, server API version: 1.19)

You likely need to upgrade your Docker Host to a get Docker client API compatibility. Do that with:

    `rig upgrade`

## Network timed out and can't pull container image

Example:

```bash
Pulling cache (phase2/memcache:latest)...
Pulling repository docker.io/phase2/memcache
Network timed out while trying to connect to https://index.docker.io/v1/repositories/phase2/memcache/images. You may want to check your internet connection or if you are behind a proxy.
```

Try restarting the Docker Machine: `rig restart`

## Started machines may have a new IP address

Example:

```bash
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
[WARN] Docker daemon not running! Trying again in 3 seconds.  Try 1 of 10.
...
[WARN] Docker daemon not running! Trying again in 3 seconds.  Try 10 of 10.
[ERROR] Docker daemon failed to start!
```

The Docker Host (probably called dev) has it's IP changed. The generated TLS Certs are no longer valid and must be regenerated.

Possible causes or relations/patterns:

* Another VM was started in VirtualBox.
* Machine went to sleep and somehow caused issues with the running VM.

Fix:

Try running: `docker-machine env`

If that does not work, you should be able to start the VM directly through docker-machine: `docker-machine start dev`

Next, check the IP / TLS status by running: `docker-machine ls`

The output will likely report something akin to:

```bash
dev       -        virtualbox   Running   tcp://192.168.99.100:2376 Unknown
Unable to query docker version: Get https://192.168.99.100:2376/v1.15/version: x509: certificate is valid for 192.168.99.101, not 192.168.99.100
```

Now, regenerate the TLS Certs: `docker-machine regenerate-certs dev -f`
Devtools should now be able to start. Don't forget to run `eval "$(rig config)"` after.

## Containers Started but Service Not Available

Your Docker Host is running, the project's containers are up, and command-line operations work fine. Why can't you view
the site in your web browser?

### DNS Services

The DNS services that Outrigger spins up may not be working. Run `docker ps` and see that you have a dnsdock and dnsmasq container.

If those services are not running, try `rig dns` to bring them up, or a full `rig restart` if that does not work.

`rig dns-records` is also useful to see what containers have registered names.

### DNS Configuration

It is also possible that the DNS services are running for your environment, but somehow the configuration is wrong. Run
`rig dns-records` and make sure your project has an entry. If not, you may need another restart, or perhaps you are
missing `com.dnsdock.name` and `com.dnsdock.image` labels in your docker-compose.yml?

### Slow-starting Services

Some services, such as Apache Solr or Varnish, can take longer to start up than Apache and PHP-FPM. As a result you might
load the browser so fast that not all services are available, which in the case of a proxy may prevent the page from
loading at all. Wait a short time and try reloading the page.

### Failed Health Checks

Some services such as Varnish depend on others to operate, and have built-in health checks to verify the other service is operating.

If such a health check fails, there could be two problems:

1. The internal DNS routing between Docker containers is broken. Make sure the
configuration of your services is correct.
1. The dependency (e.g., Apache behind Varnish) is not yet up and running when Varnish performs its checks.

In either case, you can often repair the problem by performing a clean restart of the broken service.

```bash
docker-compose stop proxy
docker-compose rm -f proxy
docker-compose up -d proxy
```

Your other services should already be up and functional when this is done, so the health check will not fail on account of (2).

> **Checking on Varnish Health**
>
> If you suspect Varnish may be failing, run `docker exec -it [VARNISH_CONTAINER] varnishlog` and scan for VCL compilation errors.

## Service Became Non-Responsive

Sometimes a service locks up. Apache stops serving results, Solr stops indexing or responding to search queries. These
things happen on servers all the time. It may even happen more often on Docker, especially since we are now using more
"infrastructure" in our local environments.

Docker is meant to easily sandbox these problems from the rest of your machine, and to easily resolve these problems by
allowing you to dump the problem and start over fresh very easily.

Hard reset on a service (as describe above), is very much like a power cycle:

```bash
docker-compose stop [BROKEN_SERVICE]
docker-compose rm -f [BROKEN_SERVICE]
docker-compose up -d [BROKEN_SERVICE]
```

Sometimes data for a service is volume mounted from outside the container. This persistence is good when the data is
healthy, but can be really bad if the data is part of the problem (e.g., broken lockfiles).

```bash
docker-machine ssh [MACHINE_NAME]
rm -Rf /data/[PROJECT]/[DIRECTORY_FOR_SERVICE]
```

Remember that your service may have configuration or data in another service, you may need to perform this operation
against multiple containers (e.g., Solr)

## NFS Conflicts with other Environments

Running the Devtools VM in conjunction with other development environments (such as Vagrant) can sometimes cause conflicts
with volume mounts. This is due to the fact that the Devtools VM mounts the entirety of `/Users` into the VM on OS X.
Additional NFS mounts from other environments that target subdirectories of `/Users` will fail.

The easiest workaround is to keep projects that use non-Devtools environments like Vagrant in a directory *outside* of
`/Users`, such as `/opt`. Alternatively, you can run a single VM at a time and manually clear out `/etc/exports` prior
to switching environments/projects.

## Migrating Vagrant boxes outside of /Users

If you'd like to move your vagrant boxes outside of your home directory, perform the following steps:

1. Choose a destination folder.  We'll be using /opt/vms for this example. Insure the destination has enough free space.
Typically this is not a problem because our Macs are one single partition.
1. Make the new directory, and ensure your userid owns it:  `sudo mkdir /opt/vms; sudo chown -R userid:userid /opt/vms`
1. Add `export VAGRANT_HOME=/opt/vms` to ~/.bash_profile
1. Move your vagrant folders over to /opt/vms.
1. edit /etc/exports, updating any vagrant mount point to use the new location.  (example: /Users/userid/vagrant/projectx/cms)
1. While a reboot isn't necessary, it'll help to make sure nothing is running and using the old mount points.
1. cd into your new vagrant directory, and do a vagrant up.

## Permission Errors During Container Startup

NFS mounts from the host volume into a container may require a specific UID mapping in order for proper permission setup within the container. SSH key volume mounts are common in project build containers. If there's a problem with permissions for host files or directories being mounted inside the docker-machine via NFS, it could be a UID mapping problem. One way this problem manifests itself is, given a mount entry in a .yml file like:

    volumes:
      - ~/.ssh/id_rsa:/root/.ssh/id_rsa

When starting the container, you may see an error like:

    ERROR: Cannot start service cli: error while creating mount source path '/Users/username/.ssh/id_rsa': mkdir /Users/username/.ssh/id_rsa: permission denied

Verify that the UID of the OSX directory is mapped correctly in `/etc/exports`.  In the host operating system (example is for OSX):

    $ id
    uid=502(username) gid=20(staff) groups=20(staff),...

In /etc/exports:

    # docker-machine-nfs-begin machine-name
    /Users 192.168.99.100 -alldirs -mapall=501:20  # <-- incorrect
    # docker-machine-nfs-end machine-name

The -mapall arguments need to match your uid and gid from the id command.  If you need to change /etc/exports (in the host), edit it, save the changes, then restart the `nfsd`:

    rig stop
    sudo nfsd restart
    rig start

## Failures during project sync startup

The initial sync process involves scanning your codebase to assemble a list of all the files it contains. For larger codebases or slower computers this process
may not complete before a timeout is reached. The timeout attempts to ensure that if there is a problem that prevents the unison container or local process from
starting you don't wait without realizing there is an issue. To see if you are getting an erroneous message message you can look to see if a local unison process
and a unison container for your project eventually start.

The easiest check can be to run the rig project sync command with the initial-sync-timeout option set to a large value such as 300 seconds.

If you want to dig deeper, look for a local unison process on your host machine with your project sync volume name as a log file. For example:
`ps aux | grep myproject-sync.log`. There should also be a docker container running with a name like myproject-sync. If both of those are present it is possible
that a timeout that is too short is affecting you.

## Identifying and troubleshooting sync issues

When the unison filesystem based syncing stops working it unfortunately does so silently. If the
files your container is using don't seem to be updating when you think they should it may be that
the syncing has stopped working. This has typically been observed happening when putting the host
machine into sleep mode. It may also happen if something causes either side of the syncing
process to crash due to a very high rate of filesystem changes. Simply restarting the sync usually
resolves the issue.

As of version 2.2.0, Outrigger has commands to help determine the state of the filesystem syncing a
as well as commands that facilitate scripting and cleanup. See the [Additional Sync Commands section
of Filesystem Sync](../project-setup/filesystem-sync.md#additional-sync-commands) for information
about these commands.
