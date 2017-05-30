# New Projects

To setup new projects in Outrigger, we have a command `rig project create` which will coordinate all of the various operations 
required to get the proper configuration in place. The `project create` command runs a collection Yeoman generators, 
prompting for input and outputting a collection of Docker Compose and Outrigger configuration.

## Project Create command

The `project create` command has the following usage:

```bash
rig project create [--image image:tag] [type] [args]
```

The `project create` command run without specifying the `--image` flag will run using the [Outrigger Generator image](https://hub.docker.com/r/outrigger/generator/).
Additionally, if no `[type]` is specified the default `type` is `outrigger-drupal`.  Other options for `[type]` on the default
`outrigger/generator` images are `gadget` and `pattern-lab-starter`.

Documentation for the currently supported types and related args can be found here:

* [outrigger-drupal](https://github.com/phase2/generator-outrigger-drupal)
* [gadget](https://github.com/phase2/generator-gadget)
* [patern-lab-starter](https://github.com/phase2/generator-pattern-lab-starter)


!!! note "Specify a Generator Image"
    If you want to use a different image other than `outrigger/generator:latest` you can specify it with this `--iamge` flag.
    This `outrigger/generator` (or other image used in its place) expects to work on a path `/generated` within the container. 
    The `project create` command mounts the current working directory as `/generated` within the container.

