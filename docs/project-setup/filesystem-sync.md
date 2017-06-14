# Filesystem Sync

Outrigger uses [Unison](https://github.com/bcpierce00/unison) to provide high-performance, bi-directional file synchronization between your local operating system and Docker containers. This is not the only means of sharing code or data with application containers, but it is the recommended approach for sharing files that must also be efficiently available to developers and their local tools (e.g., IDEs).

With Unison, all the files from the root of your project are synced via a dedicated container and exposed as a named [Docker Volume](https://docs.docker.com/engine/tutorials/dockervolumes/) to the rest of your application. This volume is used as the file "source" for a volume mount to allow nearly native filesystem performance and features.

For more on why Outrigger introduced Unison, jump down to [Why Sync Instead of NFS?](#why-sync-instead-of-nfs)

## Setting Up Sync for Your Project's Containers

To setup `unison` sync for your containers you will need to [reference an external volume](https://docs.docker.com/compose/compose-file/#volume-configuration-reference)
in your `docker-compose.yml` and use that external volume in the mount specification for any services that need to
reference the same set of files.

Since volumes are global within your Docker host, the name of this external volume should be namespaced to your project. The convention is to use `projectname-sync` as the volume name. Using the same volume will ensure all containers stay in sync. This includes specifying that volume for your [build container](/common-tasks/using-the-build-container), if you are using one (typically defined in build.yml).

### Add the volume to your docker-compose file

Note that volume definitions are available as of docker-compose schema v2. `volumes` are a root-level property in the docker-compose schema and should not be nested below services.

```yaml
volumes:
  projectname-sync:
    external: true
```

!!! important "Note about volume name"
    If you optionally specified a volume name to the `project sync:start` command or configured a name in your [Outrigger Project Configuration](./project-configuration.md) file the name in this volumes section must match the one specified.
    The reason why we don't exclusively grab the volume name from the compose file is that this volume may be
    managed outside of Outrigger by another tool. We also want to support named volumes that don't precisely follow
    the `*-sync` convention described here.

### Use the external volume as your local mount point

```yaml
services:
  build:
    image: outrigger/build:php71
    volumes:
      - projectname-sync:/var/www
```

## Managing the File Sync Process

The file synchronization is managed by the rig cli via `rig project sync:start` and `rig project sync:stop`.

### Configuring the Sync Volume Name

The name of the sync volume can be set in a variety of ways. It is determined using the following precedence:

1. Argument provided to the `project sync:start` command
2. Specified in the `sync` -> `volume-name` property of your [Outrigger Project Configuration](./project-configuration.md)
3. In your `docker-compose.yml` file, specified as an `external` volume with a name of the pattern `*-sync`
4. If not specified anywhere else, it will use the name of the current project folder with `-sync` appended

We recommend using approach #2 or #3 to ensure consistency across all environments. Approach #3 is most often used as it requires the most minimal configuration additions. If you use approach #2, make sure a volume of the same name is also defined as part of your docker or docker-compose configuration. Approach #2 is useful when a need exists to restrict which files are synced or the name of the volume can not follow the `*-sync` naming pattern. An example of limiting the synced files can be found in the sample in the [Outrigger Project Configuration](./project-configuration.md) documentation.

### Start it up

From your project root run the command `rig project sync:start` (there is also an alias `rig project sync`) to get going.
The initial sync can take a few seconds depending on the size
of your project folder. You will see a progress indicator that will let you know when the initial sync is finished and
things are ready to use.

!!! important "Workflow Change for Unison-using Projects"
    Projects that use this file syncing approach will require execution of this command as a new step before any other docker commands can be run.

### Cleaning up

Since the `project sync:start` command starts another container to manage the volume syncing, we have a command `project sync:stop`
that will clean up that running container for you. It discovers the volume / container name the same way `project sync:start`
does.

When you are done for the day, or for that project, run `rig project sync:stop` from the project root to clean up any running
`unison` containers for that project.

## Skipping File Sync on Linux Environments

Linux environments have native filesystem performance without anything as complex as file sync, because they do not need to use a virtualization layer to have access to the [Linux Kernel's container support](https://jvns.ca/blog/2016/10/10/what-even-is-a-container/).

For this reason, any environment running on Linux may want to skip the overhead of the synchronization process.

If you have this use case, create a  docker-compose.override.yml file to carry the external volume declaration, as described in the [official documentation for sharing docker-compose configuration](https://docs.docker.com/compose/extends/#multiple-compose-files). Unlike that documentation, on Linux environments you simply run as `docker-compose -f docker-compose.yml` and the configuration override will not be applied. 

### Keeping File Sync on non Linux Environments

Non Linux environments should use commands without the `-f` flag, as in `docker-compose [desired-command]`. When doing this, the override file will be automatically merged with the docker-compose.yml file. Any time the `-f` argument is used to `docker-compose` the `docker-compose.override.yml` file will need to explicitly added as well. This is most frequently needed when using a build container on a non Linux environment where the command should look like `docker-compose -f build.yml -f docker-compose.override.yml run --rm cli`.

### Example Override Files

#### Example docker-compose.yml

```yaml
services:
  www:
    image: outrigger/apache-php:php71
    volumes:
      - .:/var/www
```

#### Example docker-compose.override.yml

```yaml
services:
  www:
    volumes:
      - projectname-sync:/var/www
volumes:
  projectname-sync:
    external: true
```

## Why Sync Instead of NFS?

Outrigger provides easy, [NFS-based filesystem mounts](/project-setup/key-concepts) to share code and files with your project containers. However, it is slow when writing, reading, or scanning thousands of files when compared to native filesystem performance.
Some modern tool kits and package managers favor large numbers of small files and libraries so maintaining high performance
of a build becomes challenging when your code base is on a volume mounted NFS share.

NFS also has the downside of not propagating filesystem modification events. This can be problematic if you want to run
a process in your container to rebuild or update compiled project assets when modification notifications are triggered
due to local file editing in your IDE.

Given the current state of the art of Virtual Machines and filesystems, these types of operations are better supported by bi-directional file syncing
(which includes filesystem notifications).
