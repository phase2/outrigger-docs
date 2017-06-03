# New Projects

To setup new projects in Outrigger use the command `rig project create` to coordinate all of the various operations 
required to get the proper configuration in place. The `project create` command runs a collection of Yeoman generators, 
prompting for input and outputting a collection of Docker Compose and Outrigger configuration.

## Project Create command

The `project create` command has the following usage:

```bash
rig project create [--image image:tag] [type] [args]
```

Running the `project create` command without specifying the `--image` flag will run using the [Outrigger Generator image](https://hub.docker.com/r/outrigger/generator/).
Additionally, if no `[type]` is specified the default `type` is `outrigger-drupal`.  Other options for `[type]` on the default
`outrigger/generator` image are `gadget` and `pattern-lab-starter`.

Documentation for the currently supported types and related args can be found here:

* [outrigger-drupal](https://github.com/phase2/generator-outrigger-drupal)
* [gadget](https://github.com/phase2/generator-gadget)
* [patern-lab-starter](https://github.com/phase2/generator-pattern-lab-starter)

!!! note "Specify a Generator Image"
    If you want to use a different image other than `outrigger/generator:latest` you can specify it with the `--image` flag.
    Generator images should expect to output their work at path `/generated` within the running container.
    The `project create` command mounts the current working directory at this path in order to capture generated
    output to your host machine. All `[type]` and `[arg]` options are passed to the `docker run` command
    launching the container for the specified image.
