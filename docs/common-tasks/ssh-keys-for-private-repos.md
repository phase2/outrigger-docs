# SSH Keys for Private Repositories

## Forward/Import your SSH Key into the build container to clone private repositories

Certain containers like Jenkins and build may need your private key. This is supported by importing your 
private key into the container via a volume mount.

To get your private key into the build container, volume mount your key into the container at `/root/.ssh/outrigger.key` 
and it will be processed accordingly.

`~/.ssh/id_rsa:/root/.ssh/outrigger.key`

## Passphrase Protected Keys

For interactive operations, such as use of the build container, passphrase protected
keys do not pose a challenge as you can enter the passphrase when prompted. Services
like Jenkins, however, can prove problematic. A solution to this is to use a container
running an SSH agent with your key added and unlocked. Your container will be able
to connect with the SSH agent via the use of a shared volume and the SSH_AUTH_SOCK
variable that points to it.

### SSH Agent Container

The following is an docker compose file, ssh.yml, with services defined to help run
the necessary container for the agent and add your desired keys. It is set up for a
single SSH agent to share with all of your projects. If you prefer project specific
SSH agents, remove the `external: true` declaration in all sample files.

In order to have a single agent for all projects, first create a named volume
using `docker volume create --name=ssh`. This only needs to be done once and you'll
be prompted to do so if you forget.

```
# ssh.yml
# Provides a service for the ssh-agent and a service for adding keys to it
#
# Agent will need to be restarted and have key(s) re-added to it any time your
# docker host is restarted.
#
version: '3.2'

services:
  # use via docker-compose -f ssh.yml up [-d] ssh-agent
  ssh-agent:
    container_name: ssh-agent
    image: whilp/ssh-agent:latest
    volumes:
      - ssh:/ssh

  # use via docker-compose -f ssh.yml run --rm ssh-add $HOME/.ssh/name_of_desired_key
  ssh-add:
    image: whilp/ssh-agent:latest
    entrypoint: [ "ssh-add" ]
    volumes:
      - ssh:/ssh
      - $HOME:$HOME

volumes:
  ssh:
    external: true
```

### Connecting to the SSH Agent from other containers

Any container you want to connect to the SSH agent needs to set the environmental
variable SSH_AUTH_SOCK to /ssh/auth/sock and mount the ssh volume at path /ssh.

Here is an example ssh.overrides.yml file which shows how to add SSH agent access
to services from the [build container](using-the-build-container.md).
To use this file run `docker-compose -f build.yml -f ssh.overrides.yml run --rm DESIRED_SERVICE`

!!! note "Overrides and extends"
    Overrides do not carry through to services extending an overridden service.
    The example below demonstrates that one needs to define the override for each
    service where it should apply even though in a typical build.yml file
    the cli service is a descendant of the base service. 

```
# ssh.overrides.yml

version: '2.1'

services:
  cli:
    volumes:
      - ssh:/ssh
    environment:
      SSH_AUTH_SOCK: /ssh/auth/sock
  base:
    volumes:
      - ssh:/ssh
    environment:
      SSH_AUTH_SOCK: /ssh/auth/sock
volumes:
  ssh:
    external: true
```

The SSH agent configuration can also be placed into your build.yml file directly
if it is relevant for all team members.

### More Resources

Additional information, including ways to have non-root container processes access
the authorization socket can be found in the [whilp/ssh-agent README](https://github.com/whilp/ssh-agent) image.
