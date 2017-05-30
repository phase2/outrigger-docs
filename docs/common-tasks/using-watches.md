# Using Watches

TODO: Rework to better explain that NFS is there and easy, but not notifications. Unison requires some setup and moving 
pieces but supports full filesystem notifications and two way syncing. If you need file system notifications, you much 
use unison.

Often we need `watches` running inside of our containers. This could be for webpack, grunt, nodemon, etc.  One of the 
main challenges with the NFS mounts used in Outrigger is that they do not forward filesystem notifications across the 
NFS mount and into containers, so we need to facilitate that.

`rig` now supports unison based file syncing between the local filesystem and a volume shared with a container. This
setup allows multi-directional file syncing as well as full support for all filesystem notifications. 

See [Filesystem Sync](../project-setup/filesystem-sync.md) for details on how to setup Unison volumes.
