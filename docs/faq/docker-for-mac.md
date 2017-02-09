# Docker for Mac Support

Docker for Mac is a native hypervisor implementation of Docker that does not rely on a virtual machine provided by Docker 
Machine.  It is new with some limitations and potential conflicts with Outrigger. We will highlight the path to a peaceful 
coexistence. 

## Using only the Docker for Mac Binaries

Docker for Mac provides `docker` and `docker-compose` binaries in `/usr/local/bin`.  This install location conflicts with
the binaries provided by Homebrew, but Outrigger can use these binaries too.  If you have installed Docker for Mac first, 
you will get errors when trying to install the `docker` and `docker-compose` binaries via Homebrew. Those errors are fine, 
as long as when all is said and done `docker`, `docker-compose`, and `docker-machine` are all available on your `$PATH`.

To use the binaries with your Outrigger VM you simply need to run the `eval "$(rig config)"` command to setup your
Docker environment.  This will point the `docker` to your VM.  You will need to do this in every shell that you want to 
use the Outrigger VM.  If you then need to transition back to using the Docker for Mac hypervisor, you will need to unset 
the Outrigger `docker` configuration. Use `eval "$(docker-machine env -u)"` to unset all `DOCKER_*` environment variables. 

If you need to do this often, we recommend setting up aliases to set/unset the environment vars.

```bash
alias re='eval "$(rig config)"'
alias ru='eval "$(docker-machine env -u)"'
```

## Using the Docker for Mac Binaries and Hypervisor

To use the Docker for Mac Hypervisor follow the basic instructions below.

!!! important "Docker for Mac is less performant (early 2017)"
    The FUSE driver they use to share your host filesystem into the hypervisor/containers is not nearly as performant as
    the NFS mount we use with the Outrigger VM.  Therefore, operations that do either a lot of filesystem reading (like 
    directory scan) or writing (like restoring DB dumps) will take much longer to perform.  It is advisable to use the
    Outrigger VM / VirtualBox for the time being simply for performance reasons.
    
!!! note "Networking limitation when using the Docker for Mac Hypervisor"
    There are some [known limitations](https://docs.docker.com/docker-for-mac/networking/#/known-limitations-use-cases-and-workarounds)
    with the way networking is implemented with Docker for Mac.  Most notable we can not directly access containers on 
    their native IP address due to the lack of the docker0 bridge network that exists in the Docker Machine VM implementation.
    These limitations inhibit the fluid environment that Outrigger enables and as such is not natively supported (yet). We
    have done our best to highlight the issues and some of our ideas for workarounds below. Let us know how it goes if you
    venture down this path.

### Setup /data

Outrigger makes a convention out of a `/data` directory within the VM.  To provide the same directory to Docker for Mac, 
create a `/data` directory in the root of your Mac filesystem.  You may need to open up the permissions so the container(s)
can write to it.

```bash
sudo mkdir /data && sudo chmod 777 /data
```

Once you have created that directory, go into Docker for Mac Preferences > File Sharing and add `/data` to the list of 
shares and *Apply & Restart* Docker for Mac.  With this `/data` directory setup your Compose files should work on either 
the Docker for Mac Hypervisor or the Outrigger VM.

### Accessing your Container Services

Docker for Mac does not provide a mechanism to route network traffic directly to your containers, they only support 
publishing (binding) ports from your running containers to your host.  This means that all services are accessed on 
localhost (127.0.0.1) and share the same port space. (Two web server can't both publish to port 80). The approach we think 
makes sense (but don't actively support) is:
 
* Run **jwilder/nginx-proxy** as your only service that binds to port 80
* Start all of your web containers with the `VIRTUAL_HOST` environment variable used by nginx-proxy to specify it's domain name
* Resolve the domain name from the above `VIRTUAL_HOST` variable in one of two ways
    * Edit your `/etc/hosts` file and point the domain name at `127.0.0.1`
    * Use dnsmasq
        * Make sure dnsdock is not running
        * In `/etc/resolver/vm` have `nameserver 127.0.0.1`
        * Run a dnsmasq container, bind it to port 53, configured to resolve all **.vm** addresses to `127.0.0.1`

## Uninstalling Docker for Mac

Since Docker for Mac and Homebrew both install the Docker binaries in to `/usr/local/bin` after you uninstall Docker for Mac 
your Outrigger environment wont work.  You may see an error like `command not found: docker`. The Docker for Mac 
installation overwrote the Homebrew based Docker installation, but brew still believes they are installed, so you'll 
need to unlink and re-link.

```bash
brew unlink docker && brew link docker
brew unlink docker-compose && brew link docker-compose
brew unlink docker-machine && brew link docker-machine
```