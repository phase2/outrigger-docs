# Filesystem Sync with Unison

The default NFS filesystems that Outrigger provides are very easy to setup and easy to share code and other files into
your project containers, but NFS can be slow when writing, reading, or scanning thousands of files when compared to
native filesystem performance. This can be a particular challenge when building your code base from a package manager onto
a volume mounted NFS share.

It also has the downside of not propagating filesystem modification events, which can be problematic if you wanted to have
a process ina  container monitor yor files and run a process to rebuild or update when modification notifications are triggered.

In order to better support these types of operations, Outrigger provides support for multi-directional filesystem syncing,
that also supports filesystem notifications, called `unison`.  Your project root get created as a Docker Volume and is
sync'd via unison in to a sync container.  This volume is then used as your mount point instead of the NFS share.


TODO: Document `rig project sync:start` and `.outrigger.yml` and `docker-compose.yml` configuration
TODO: Document volume name discovery