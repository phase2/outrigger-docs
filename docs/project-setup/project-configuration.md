# Project Configuration

Projects can have a local `.outrigger.yml` file included in their project to define the `rig project` commands. This can
be done to streamline the developer experience of the project, or to capture project specifics / intricacies in a place that
is consistent for all team members.

## File Location

The configuration file is typically named `.outrigger.yml` and is normally placed at the root of a project directory tree.

You may specify a $RIG_PROJECT_CONFIG_FILE environment variable to override this which can be useful for running rig project
commands from different directories.

## Features

### Project Scripts

Much like composer and npm, rig project supports the creation of project-standardized scripts in this configuration file.

* Scripts are executed relative to the location of the configuration file.
* The `bin` property is prepended to your $PATH variable, simplifying the script configuration.
* Each "script" may be made up of a series of steps, if any step fails remaining steps will not be executed.
* When you run a script (such as `rig project run:up`) you can include additional parameters.
  Such additonal parameters will be appended to the final step of the script.

Commands are defined in rig project based on the script ID prefixed by `run:`, so a command such as
`up` is primarily available as `rig project run:up`.

Any script may specify a single alias to shorten this. An alias such as 'up' would change this to `rig project up`.

### Configure Filesystem Sync

As described in [Filesystem Sync](./filesystem-sync/), rig project brokers the synchronization process.

You can tailor this process to specify the name of the Docker volume or set files or directories to be
skipped by this process as a performance optimization.

None of this configuration is required.

## Configuration File

Below is a sample configuration file with comments to describe the various configuration options.

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
from the `outrigger-drupal` generator, for an example of a detailed `.outrigger.yml` file.
