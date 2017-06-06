# Project Configuration

Projects can have a local `.outrigger.yml` file included in their project to customize the `rig project` commands. This can
be done to add to the developer experience of the project, or to capture project specifics / intricacies in a place that
is consistent for all team members.

`.outrigger.yml` is usually placed in the project root and can contain configuration to customize the `rig project run` 
and `rig project sync` commands.  Below is a sample configuration file with comments to describe the various configuration options.

```yaml
# Required version so we can ensure compatibility
version: 1.0

# This is prepended to the $PATH for any commands referenced in the scripts section.
bin: ./bin:./node_modules/.bin:./vendor/bin

# Project Scripts
# These can be run via 'rig project run:<key>'
# If you specify an alias, you can run 'rig project <alias>'
scripts:

  # This will be `rig project run:up`
  up:
    # Or with an alias, `rig project up`
    alias: up
    # Description is used for help when running `rig project help` or `rig project run help`
    description: Start up operational docker containers and filesystem sync.
    # These are the various run steps, they will be concatenated together into a single command with '&&'
    run:
      - rig project sync:start
      - docker-compose up -d

  stop:
    alias: down
    description: Halt operational containers and filesystem sync.
    run:
      - docker-compose stop
      - rig project sync:stop

# This controls configuration for the `project sync:start` command.
sync:
  # This is the name of the external volume to use (optional). Needs to match volume name in Docker Compose. 
  volume: project-sync
  # These ignores will be added to unison and not synced between local and project volume
  ignores:
    - "Name .git"
    - "Path build/tmp"
    - "Regex full/path/to/*.json"
    
```

See [this Outrigger Project configuration file](https://github.com/phase2/generator-outrigger-drupal/blob/master/generators/environment/templates/outrigger/outrigger.yml), 
from the `outrigger-drupal` generator, for an example of a detailed `/outrigger.yml` file.