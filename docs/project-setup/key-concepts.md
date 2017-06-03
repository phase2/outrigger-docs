# Common Setup

Before you begin there are two aspects of Docker and Outrigger setup that you must be aware of.
These relate to how you share your code with your running Docker containers and how you ensure that
any data you need to persist between executions of your containers is preserved. The examples you
see throughout this documentation demonstrate configurations that take these aspects into
consideration.

## Project Code

The filesystem within containers is **ephemeral!** Any changes made there **do not persist** if the 
container is restarted. (See "Persistent Data Volume" below for a storage area that is preserved.)

In typical Docker images the code is built **directly into the container** at the `/code` path. This 
is a great mechanism that allows the container to be "self contained" (pun intended), immutable and 
not need any checkout/file system. You also know when you run a container exactly what code is in 
there because you generally don’t change the code unless you rebuild the image.  

For development purposes, however, this is problematic because it is burdensome to rebuild an image 
for each code change. To solve this, there are a few available approaches you can take.

### Approach 1: NFS Filesystem

Outrigger sets up an NFS filesystem from your Host Machine to the Docker host that you can leverage.
Using this, you can easily mount your project code from the Host Machine into the running 
container, effectively overriding the files built directly into the image. The running container is 
then using the local project file system for the overridden paths rather than the file system built 
into the image.  This allows a developer to use an IDE and edit code directly on the local file system 
of the Host Machine, but execute that code within the environment of the running container.

!!! note "Note on Project Code Location"
    Your project code **must** be located somewhere within your home directory (`/Users` for Macs) on 
    your local machine. This is because NFS shares your home directory into the Docker Host VM, and 
    only files on the Docker Host VM can be referenced in volume shares. 

### Approach 2: Docker Volumes and Unison

There are some drawbacks to using the NFS Filesystem so your second option is to use a 
[Data Volume](https://docs.docker.com/engine/tutorials/dockervolumes/) to hold a mirror of your
codebase and user Outrigger's [Filesystem Sync](filesystem-sync.md) capabilities to automatically
keep it in sync with your local codebase. There are several advantages to this approach detailed
on the filesystem sync page at the cost of a slightly more complex setup. Outrigger helps manage
the creation and syncing of the data volume to minimize this complexity.

## Persistent Data Volume

Outrigger maintains a data volume on the Docker Host that is mounted at `/data` into every container. 
This volume is persistent so long as you do not perform a `rig remove` operation. This ensures 
that file access on the VM is done natively for things like databases and other things where filesystem
performance matters. If you configure a container to write to this area, you should use a project and 
container based namespace to prevent conflicts as this is a shared resource. For example, you may want 
to use a namespacing method like `/data/[project name]/[environment]/[service name]` as a safe location 
to write data. Over time you may find it necessary to manage this directory and delete old project
data. This can be done through use of `docker-machine ssh dev` and then direct manipulation of `/data`.

Any code from your local directories is directly shared in to the Docker Machine VM via NFS at `/Users`. 
This means that even if you destroy and re-create your Docker Host, your code will be safe since it 
lives on the Host Machine.

!!! note "FILE CHANGES WITHIN A CONTAINER"
    Any files that are generated or changed within a running container that you want to persist after the 
    container is stopped **should be put onto a persistent volume local machine**.  
    A Docker container represents immutable infrastructure, the files on the image 
    are able to be changed at runtime but typically do not persist. When the container is stopped and 
    run again it "boots" the files that were built into the original image. The volume at `/data` (mentioned 
    above) is a persistent volume that you can use and Docker Data Volumes represent another.


    Outrigger helps maintain the contents of the `/data` directory between upgrades of your Docker
    Host Virtual Machine though occasional maintenance to remove old project data will be needed.
    Docker data volumes do not persist between virtual machine upgrades but may be easier to manage
    using `docker volume` commands.
