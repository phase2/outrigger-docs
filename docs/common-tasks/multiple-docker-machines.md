# Multiple Docker Machines 

There are special cases in which you may want to have multiple Docker Hosts running, or use a Docker Machine from 
another client/project. One example of this might be to test the performance differences between VirtualBox and xhyve.  

For these cases, `rig` takes a `--name` flag that will allow you to specify name of the Docker Host. For all commands 
this name defaults to `dev` but you can configure this name if needed. Also, if you don't want to specify the Docker 
Host name on every command you can specify the `RIG_ACTIVE_MACHINE` environment variable to be Docker Machine name of 
the VM you care to interact with.  This way you can set it once and forget about it.  Unless otherwise specified, the 
name defaults to `dev` for all commands.
