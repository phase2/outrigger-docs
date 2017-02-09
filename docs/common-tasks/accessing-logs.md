# Accessing Logs

## Viewing logs from my container services

When you start a your containers via `docker-compose up` all of the defined services will start in the foreground. All 
log output to stdout/stderr within each container will be output the console.  Each entry will be prefixed with the name 
of the running container to identify the source of the log message.

If log output is not coming directly to the console you can 
[SSH into the container](../common-tasks/ssh-into-a-container.md) and browse the file system for logs. Most common 
services will provide some log output in `/var/log`.