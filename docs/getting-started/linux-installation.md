# Linux Installation

When running Docker containers on Linux, it is not necessary to run the Docker Machine VM. The instructions here describe 
how to run Outrigger projects on Linux.

## Install Outrigger

* Go to the [Outrigger Releases](https://github.com/phase2/rig/releases) page and download the `.rpm`, `.deb`, 
or grab the `linux-amd64.tar.gz` binary directly and install it in `/usr/local/bin`

## Linux Requirements

1. For automated DNS resolution, the use of one of two options to resolve DNS queries to the dnsdock container

## Linux installation on Fedora/Centos

1. [Install Docker for Fedora](https://docs.docker.com/engine/installation/linux/fedora/) or 
[Install Docker for CentOS](https://docs.docker.com/engine/installation/linux/centos/)
2. [Install Docker Compose](https://docs.docker.com/compose/install/)
3. Set the DNS configuration for Docker
    - We need to add Docker daemon configuration
    - `sudo vi /etc/docker/daemon.conf`
    - In that file put (or integrate) the following:    
```json
{
    "dns": [ "172.17.0.1" ]
}
```
4. Set up the docker0 network as trusted
    - `sudo firewall-cmd --zone=trusted --add-interface=docker0 && sudo firewall-cmd --zone=trusted --add-interface=docker0 --permanent`
5. Restart the docker daemon
    - `sudo systemctl restart docker`

## Linux installation on Ubuntu/Debian

1. [Install Docker for Ubuntu](https://docs.docker.com/engine/installation/linux/ubuntu/) or 
[Install Docker for Debian](https://docs.docker.com/engine/installation/linux/debian/)
2. [Install Docker Compose](https://docs.docker.com/compose/install/)
3. Set the DNS configuration for Docker
    - We need to add Docker daemon configuration
    - `sudo vi /etc/docker/daemon.conf`
    - In that file put (or integrate) the following:    
```json
{
    "dns": [ "172.17.0.1" ]
}
```
4. Restart the docker daemon
    - `sudo systemctl restart docker`

## Automated Linux DNS configuration options

Outrigger will help automate the setup of DNS (via `rig start` and `rig dns`) if you are using one of the following options for name resolution

1. [dnsmasq via NetworkManager](https://wiki.archlinux.org/index.php/dnsmasq#NetworkManager)
2. [libnss-resolver](https://github.com/azukiapp/libnss-resolver)

## Manual DNS resolution options

### DNSDock as main resolver

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
  aacebedo/dnsdock:v1.16.4-amd64 --domain=vm
```

1. Configure 172.17.0.1 as your first DNS resolver in your network configuration. The method for doing this may differ 
based on whether you are using a desktop environment or running Linux on a server, but that nameserver should end up as 
the first 'nameserver' line in your `/etc/resolv.conf` file.
2. Running dnsdock as a service
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
ExecStart=/usr/bin/docker run --rm --name dnsdock -v /var/run/docker.sock:/var/run/docker.sock -l com.dnsdock.name=dnsdock -l com.dnsdock.image=outrigger -p 172.17.0.1:53:53/udp aacebedo/dnsdock:v1.16.4-amd64  --domain=vm
ExecStop=/usr/bin/docker stop dnsdock
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

3. Ensure Docker is registered with systemctl
    - `systemctl enable docker`
4. Register dnsdock service: 
    - `systemctl enable dnsdock && systemctl daemon-reload`
5. Now you can start/stop the dnsdock service with 
    - `systemctl [start|stop] dnsdock`
6. You can also check its status 
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
        - `ping test.outrigger.vm`
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
  aacebedo/dnsdock:v1.16.4-amd64 --domain=vm
  
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
