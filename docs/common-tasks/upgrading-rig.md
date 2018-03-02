# Upgrading Outrigger

## Upgrade Outrigger CLI, rig

### Homebrew

If you installed rig via Homebrew, you can check for updates by running

```bash
brew update
brew upgrade outrigger-cli
```

### Testing your installation

Run `rig doctor` to confirm the system is in good working order.

### Upgrading your Docker Host

If your `brew upgrade` happens to also update docker, when running `rig doctor` you 
may get a warning about an incompatible Docker version in your Docker Host VM. In that case use the
`rig upgrade` command to upgrade your Docker Host VM to a compatible version.
