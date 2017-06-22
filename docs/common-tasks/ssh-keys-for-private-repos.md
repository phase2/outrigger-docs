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
running an ssh agent with your key added and unlocked. Your container will be able
to connect with the ssh agent via the use of a shared volume and the SSH_AUTH_SOCK
variable that points to it.

### SSH Agent Container

The following is an ssh.yml file that can help set up the necessary containers and
add your desired keys to it.

```
# ssh.yml
# Provides a service for the ssh-agent and a service for adding keys to it
#
# Agent will need to be restarted and have key(s) re-added to it any time your
# docker host is restarted.
#
version: '2.1'

services:
  # use via docker-compose -f ssh.yml up [-d] ssh-agent
  ssh-agent:
    container_name: ssh-agent
    image: whilp/ssh-agent:latest
    volumes:
      - ssh:/ssh

  # use via docker-compose -f ssh.yml run --rm ssh-add $HOME/.ssh/name_of_desired_key
  # use as many times as needed to add the desired keys
  ssh-add:
    image: whilp/ssh-agent:latest
    entrypoint: [ "ssh-add" ]
    volumes:
      - ssh:/ssh
      - $HOME:$HOME

volumes:
  ssh:
```

### Connecting to the SSH Agent from other containers

Any container you want to connect to the SSH agent needs to set the environmental
variable SSH_AUTH_SOCK to /ssh/auth/sock and mount the ssh volume at path /ssh.

Here is an example build.ssh.yml file which extends the build.yml file's base
container and adds the ssh agent ability.

```
build.ssh.yml
version: '2.1'

services:
  base:
    extends:
      file: build.yml
      service: base
    volumes_from:
      # note that this is using the container name so if you choose a different
      # one they must be kept in sync
      - container:ssh-agent
    environment:
      SSH_AUTH_SOCK: /ssh/auth/sock
```

### More Resources

Additional information, including ways to have non root container processes access
the authorization socket can be found in the [whilp/ssh-agent README](https://github.com/whilp/ssh-agent) image.
