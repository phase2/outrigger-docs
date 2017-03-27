# Windows Installation

!!! warning
    Windows is not yet fully supported but the following information should
    help you get a functioning environment running. Submissions of
    information that helps improve the command examples and that can be
    rolled into the automation afforded by the rig binary are welcome.

## Install VirtualBox

[VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)

## Install docker tools

You will need to install the following docker tools. It is recommended
that you use [chocolatey](https://chocolatey.org/) or an installation
mechanism which allows you to simply acquire these binaries with no
additional setup.

* docker
* docker-machine
* docker-compose

## Download the latest `rig` binary

The latest Windows binary for rig can be found at https://github.com/phase2/rig/releases.

This binary is responsible for a few tasks on the initial execution of
the `rig start` command. Information below assumes default values for
all options.

* Checking for the existence of a virtual machine instance to serve as a
docker host and creation if it does not exist. Creation is of a virtual
machine named dev using a boot2docker iso file based on the version of
the `docker` binary installed. This is a command that typically looks like:
`docker-machine create dev --driver=virtualbox --virtualbox-boot2docker-url=https://github.com/boot2docker/boot2docker/releases/download/v17.04.0-ce-rc1/boot2docker.iso --virtualbox-memory=4096 --virtualbox-cpu-count=2 --virtualbox-disk-size=40000 --engine-opt dns=172.17.42.1 --engine-opt bip=172.17.42.1/16 --virtualbox-hostonly-nicpromisc allow-all`
* Startup of created virtual machine serving as docker host.
* Ensuring proper environmental variable settings for communicating
with the running machine. See `docker-machine env dev` for needed
variables and values.
* Setup of routing rules so that all traffic destined for 172.17.0.0/16
is routed to the IP address of the running instance. This is usually
a command that looks something like:
`runas /noprofile /user:Administrator route DELETE 172.17.0.0` to clean
any lingering setup and then
`runas /noprofile /user:Administrator route -p ADD 172.17.0.0/16 $(docker-machine ip dev)`
* Setting up the /data directory within the docker host to point at a
persistent disk to allow space for non ephemeral data to be stored.
There should be a `/mnt/sda1` drive available within the virtual machine
that a data directory should be created on and then linked to via the
command `sudo ln -s /mnt/sda1/data /data`
* Instantiation of a dnsdock container to provide DNS services via the
command `docker run --restart=always -d -v /var/run/docker.sock:/var/run/docker.sock -e DNSDOCK_NAME=dnsdock -e DNSDOCK_IMAGE=dnsdock --name dnsdock -p 172.17.42.1:53:53/udp phase2/dnsdock`
* Setup of DNS services so that \*.vm addresses attempt to resolve
against the running dnsdock container. It is currently unknown how
to get Windows to accomplish this in a safe way when the containers
are not running but a work around is to use the `rig dns-records`
command to provide output for entry into the
`C:\Windows\System32\Drivers\etc\hosts` file.
* Sharing of the home directory of all of your users via NFS with the
dev instance. On OS X this is accomplished via the [docker-machine-nfs.sh](https://github.com/adlogix/docker-machine-nfs)
script. This will allow for code on your local machine to be executed
from within a running container as long as it is in your home directory.
