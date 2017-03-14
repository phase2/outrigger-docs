# Linux Installation

When running Docker containers on Linux, it is not necessary to run the Docker Machine VM. The instructions here describe 
how to run Outrigger projects on Linux.

## Linux Requirements

1. The dnsdock container, used to support automatic creation and maintenance of DNS namespace for the containers
1. Use of one of three options to forward DNS queries to the dnsdock container (see 
[Linux DNS configuration options](#markdown-header-linux-dns-configuration-options) below)

## Linux installation on Fedora/Centos

1. [Install Docker for Fedora](https://docs.docker.com/engine/installation/linux/fedora/) or 
[Install Docker for CentOS](https://docs.docker.com/engine/installation/linux/centos/)
1. [Install Docker Compose](https://docs.docker.com/compose/install/)
1. Set the DNS configuration for Docker
    - We need to modify the command that docker uses within systemd
    - `sudo mkdir /etc/systemd/system/docker.service.d`
    - `sudo vi /etc/systemd/system/docker.service.d/docker.conf`
    - In that file put something like the following:    
```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --dns=172.17.0.1
```
1. Set up the docker0 network as trusted
    - `sudo firewall-cmd --zone=trusted --add-interface=docker0 && sudo firewall-cmd --zone=trusted --add-interface=docker0 --permanent`
1. Start the docker daemon
    - `sudo systemctl start docker`

## Linux installation on Ubuntu/Debian

1. [Install Docker for Ubuntu](https://docs.docker.com/engine/installation/linux/ubuntu/) or 
[Install Docker for Debian](https://docs.docker.com/engine/installation/linux/debian/)
1. [Install Docker Compose](https://docs.docker.com/compose/install/)

**If using Upstart**

- Set the DNS configuration for dnsdock, as well as known RFC-1918 address space
    - Please note that the following command will over-write your existing Docker daemon configuration file.  Please 
    set the -dns=172.17.0.1 option manually as an alternative
    - `echo 'DOCKER_OPTS="-dns=172.17.0.1"' | sudo tee /etc/default/docker`
- Start the docker daemon
    - `sudo start docker`

**If using Systemd**

- Set the DNS configuration for Docker
    - We need to modify the command that docker uses within systemd
    - `sudo mkdir /etc/systemd/system/docker.service.d`
    - `sudo vi /etc/systemd/system/docker.service.d/docker.conf`
    - In that file put something like the following:    
```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --dns=172.17.0.1
```

## Linux DNS configuration options

### Method 1: dnsmasq via NetworkManager

This method works well with no other needed software provided that you have unfettered access to your system's 
configuration, and are using NetworkManager to maintain your networking stack

1. Add the line dns=dnsmasq to /etc/NetworkManager/NetworkManager.conf under the [main] configuration stanza. This will 
cause NetworkManager to spawn and use a dnsmasq process for all name resolution. If you already have a local configuration, 
ensure that it is not configured to start on system boot.
1. Add a single rule to direct all DNS lookups for .vm addresses to the 172.17.0.1 address.
    - `echo 'server=/vm/172.17.0.1' | sudo tee /etc/NetworkManager/dnsmasq.d/dnsdock.conf`
1. Restart NetworkManager, either through systemd, or by simply rebooting.  To restart via systemd:
    - `systemctl restart NetworkManager`
1. Run the dnsdock container

```bash
docker run -d \
  --name=dnsdock \
  --restart=always \
  -l com.dnsdock.name=dnsdock \
  -l com.dnsdock.image=outrigger \
  -p 172.17.0.1:53:53/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aacebedo/dnsdock:v1.16.1-amd64 --domain=vm
```

### Method 2: dnsdock as main resolver

This method will probably only work well if this is a fixed computer or server with a consistent single upstream DNS 
server. If you meet these criteria, you can very easily use this to set up .vm resolution for containers an delegate the 
rest to your normal DNS server.

This example assumes that the upstream DNS server for a Linux workstation is 192.168.0.1.

1. Run the dnsdock container, specifying your upstream DNS server at the end.

```bash
docker run -d \
  --name=dnsdock \
  --restart=always \
  -l com.dnsdock.name=dnsdock \
  -l com.dnsdock.image=outrigger \
  -p 172.17.0.1:53:53/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aacebedo/dnsdock:v1.16.1-amd64 --domain=vm
```

1. Configure 172.17.0.1 as your first DNS resolver in your network configuration. The method for doing this may differ 
based on whether you are using a desktop environment or running Linux on a server, but that nameserver should end up as 
the first 'nameserver' line in your /etc/resolv.conf file.

### Method 3: libnss-resolver

libnss-resolver is an app that adds Mac-style /etc/resolver/$FQDN files to the Linux NSS resolution stack to query a 
different DNS server for any .vm address.  It may be the easiest option for most installations.

There are releases for Fedora 20, Ubuntu 12.04 and Ubuntu 14.04.

1. Install [libnss-resolver](https://github.com/azukiapp/libnss-resolver/releases)
1. Set up .vm hostname resolution
    - `echo 'nameserver 172.17.0.1:53' | sudo tee /etc/resolver/vm`
1. Run the dnsdock container

```bash
docker run -d \
  --name=dnsdock \
  --restart=always \
  -l com.dnsdock.name=dnsdock \
  -l com.dnsdock.image=outrigger \
  -p 172.17.0.1:53:53/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aacebedo/dnsdock:v1.16.1-amd64 --domain=vm
```

## Running dnsdock as a service

- Create the file `/etc/systemd/system/dnsdock.service` with the following contents

```bash
[Unit]
Description=DNSDock
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill dnsdock
ExecStartPre=-/usr/bin/docker rm dnsdock
ExecStart=/usr/bin/docker run --rm --name dnsdock -v /var/run/docker.sock:/var/run/docker.sock -l com.dnsdock.name=dnsdock -l com.dnsdock.image=outrigger -p 172.17.0.1:53:53/udp aacebedo/dnsdock:v1.16.1-amd64  --domain=vm
ExecStop=/usr/bin/docker stop dnsdock
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

- Ensure Docker is registered with systemctl
    - `systemctl enable docker`
- Register dnsdock service: 
    - `systemctl enable dnsdock && systemctl daemon-reoload`
- Now you can start/stop the dnsdock service with 
    - `systemctl [start|stop] dnsdock`
- You can also check its status 
    - `systemctl status dnsdock`

## Verifying DNS is working

Once you have your environment set up, you can use the following tests to ensure things are running properly.

- `dig @172.17.0.1 dnsdock.outrigger.vm.`
    - You should get a 172.17.0.0/16 address
- `ping dnsdock.outrigger.vm`
    - You should get echo replies from a 172.17.0.0/16 address
- `getent hosts dnsdock.outrigger.vm`
    - You should get a 172.17.0.0/16 address
- Verify DNS works between containers
    - Open a shell for a second test container
        - `docker run --rm -l com.dnsdock.name=test -l com.dnsdock.image=outrigger -it alpine sh`
    - From its prompt: 
        - `ping dnsdock.outrigger.vm`
        - You should get echo replies from a 172.17.0.0/16 address

## Troubleshooting

### `bind: address already in use`

Make sure you are not running a service which binds all mapped IPs.  For example,

```bash
> docker run -d \
  --name=dnsdock \
  --restart=always \
  -l com.dnsdock.name=dnsdock \
  -l com.dnsdock.image=outrigger \
  -p 172.17.0.1:53:53/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aacebedo/dnsdock:v1.16.1-amd64 --domain=vm
  
  6aa76d0df98ede7e01c1cef53f105a79a60bab72ee905a1acd76ad57d4aeb014
  docker: Error response from daemon: driver failed programming external connectivity on endpoint dnsdock
  (62b3788e1f60530d1f468b46833f0bf8227955557da3a135356184f5b7d57bde): Error starting userland proxy: listen
  udp 172.17.0.1:53: bind: address already in use.
```

In this case the culprit was `bind9/named` was attaching to all interfaces (on port 53)

- The solution
    - `systemctl stop bind9`
- To avoid having to stop `bind9` every session
    - `systemctl disable bind9`
- You may need to restart NetworkManager
    - `systemctl restart NetworkManager`
