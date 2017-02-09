# Using Watches

Often we need `watches` running inside of our containers. This could be for webpack, grunt, nodemon, etc.  One of the 
main challenges with the NFS mounts used in Outrigger is that they do not forward filesystem notifications across the 
NFS mount and into containers, so we need to facilitate that.

There is a command `rig watch [options] <path>`. Running this on your host will watch for changes and rsync 
notifications to files and directories under `<path>` into the Docker Machine VM (which will then provide filesystem 
notifications into the container). These are the options for `rig watch`

* `--machine <name>` Optional: Specify a machine to send events. It will default to our `dev` machine
* `--ignorefile <file>` Optional: Specify a file that contains patterns for directories/files to ignore.  Put one 
entry per line (blank lines and comments are allowed). If not specified it will look for a file named 
`.rig-watch-ignore` in the working directory and all parent directories.
