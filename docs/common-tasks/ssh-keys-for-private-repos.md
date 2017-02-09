# SSH Keys for Private Repositories

## Forward/Import your SSH Key into the build container to clone private repositories

Certain containers like Jenkins and build may need your private key. This is supported by importing your 
private key into the container via a volume mount.  

To get your private key into the build container, volume mount your key into the container at `/root/.ssh/outrigger.key` 
and it will be processed accordingly.

`~/.ssh/id_rsa:/root/.ssh/outrigger.key`
