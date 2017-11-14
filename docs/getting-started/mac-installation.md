# Installation

## Mac Installation

### Install VirtualBox

[VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)

### Install Homebrew

[Homebrew Website](http://brew.sh/)

If Homebrew is already installed, then be sure to do a `brew update`

### Tap the Outrigger repository

`brew tap phase2/outrigger`

### Install Outrigger (and dependencies)

`brew install outrigger-cli`

This will install the `rig` binary as well the Docker and other dependencies.

### Create the Docker Host

Once everything checks out, run the following command to create a new docker host.
(You will likely be prompted for your admin password)

`rig start`

Here are some configuration options available to you to customize your setup:

Options on `rig`:

* **name:** The Docker Machine name for the VM. Defaults to `dev`

Options on the `start` command:

* **--driver:** The driver to create the Docker Machine with. Choices are:
    * `virtualbox` - default
    * `vmwarefusion`
    * `xhyve`
* **--cpu-count** The number of virtual CPU you want to allocate to this VM. Defaults to 2
* **--memory-size** The size memory you want to configure for this VM, in Megabytes. Defaults to 4096
* **--disk-size** The size drive you want to configure for this VM, in Gigabytes. Defaults to 40

### Configure your shell to set Outrigger Docker environment

To configure the shell with the proper Outrigger environment, run the following command
after the docker host has started from the previous step.

`eval "$(rig config)"`

For convenience, you should make this automatic on every terminal you launch. To do that
add the following to your `.bash_profile`, `.zshrc` or equivalent:

```bash
# Support for Outrigger
eval "$(rig config)"
alias re='eval "$(rig config)"'
```

!!! note "Running rig config"
    Even with automatic execution in your shell, this command must be run in any existing
    terminal windows after `rig start` or `rig restart` commands. To support that
    see the `re` alias in the shell init file above. Just run `re` after a `rig start`

### Support for Docker for Mac

See [Docker for Mac Support](../faq/docker-for-mac.md)
