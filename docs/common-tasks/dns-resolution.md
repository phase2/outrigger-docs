# DNS Resolution

## DNS Names for your containers

Within the `labels` section of the docker-compose file you can specify labels that will control the DNS name of your 
containers.  DNS names are in the format of `[name].[image].vm` (e.g web.outrigger.vm)  

* `com.dnsdock.name` - This is the type of container. Usually something like web, db, cache, etc.

* `com.dnsdock.image` - This is generally your project name (e.g outrigger, drupal, etc.)

!!! note
    All DNS names of Outrigger containers will end in *.vm*

If you need multiple domains mapped to a container you can use the `com.dnsdock.alias` setting. It takes a full domain 
name and can take a comma separated list (e.g. otherhost.outrigger.vm or alexandria.phase2.vm). Additionally, dnsdock 
supports longer domain queries meaning if you have a service named web.outrigger.vm, you can also use 
something.web.outrigger.vm and it will resolve to the same IP address.

For all other DNS configuration options, please see the [dnsdock label documentation](https://github.com/aacebedo/dnsdock#overrides-with-docker-labels)

### DNS Forwarders

Outrigger can be configured to use additional name servers to forward DNS requests if the record cannot be resolved by dnsdock.

An example is if you connect to a VPN and need to resolve addresses to private servers within a VPN.  To enable this you 
need to configure the `RIG_NAMESERVERS` environment variable to a comma separated list of _ip:port_ 
(example: `10.10.7.2:53,8.8.8.8:53`) before running either `rig start` or `rig dns`. We suggest putting this 
env var configuration in your `~/.bashrc` or `~/.zshrc` so that it is always present when needed.  If you just need this 
temporarily, you can pass the configuration in with the `--nameservers` command line option to `rig start` or `rig dns`. 

This configuration will try each forwarder name server, in order, to resolve names until success or all name servers 
have been exhausted.

### DNS Command

There is also a `rig dns` command that will launch and configure our DNS services on any Docker Host. If you want to 
configure DNS on a Docker Host other than _dev_ be sure to specify the `--name` parameter on `rig`.

### DNS Debugging

If you want to see what containers have registered names, use the `rig dns-records` command. This command will list 
all registered container names and aliases along with the container's IP address.
