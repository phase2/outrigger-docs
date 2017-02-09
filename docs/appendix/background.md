# Background

This document covers some of the history of development tooling and explains some of the how and why of the Outrigger approach.

## Why Outrigger

Outrigger covers an entire toolbox of solutions to accelerate development and smoothly implement best practices. We
intend to use the Outrigger umbrella to help us roll these tools and approaches out to the whole company in a more
consistent fashion.

Across a wide variety of projects and clients, keeping a development, integration, staging, production and other
environments synchronized in terms of server configuration and supporting software versions and tooling has always been
a challenge. The multitude of platforms and versions of operating systems and software is only growing. In the past we
have used Vagrant in conjunction with a virtual machine (VM) for local development environments, project-based VMs on
OpenStack for integration environments and tools like Puppet to try and ensure all of those match each other and the
deployment environments.

It’s been better than doing it all by hand but it has required building additional expertise above typical system
management tools and it’s relatively inefficient to have many project-specific VMs locally. On top of that are the
various, sometimes conflicting, versions of development toolchains which the VMs don’t always address.

Outrigger is going to address this problem by providing an efficient way to have project/app-specific environments as well
as development tooling by using a concept called containerization. This lets us focus on items at a service-and-tools
level rather than at a server level and make the details of our hosting implementation more transparent to all technical
team members.

## Mapping Concepts From Previous Approaches

There are some important pieces to remember about how containerization is similar and different to concepts you may be
familiar with.

### Vagrant

Vagrant is being replaced by the combination of Docker Compose and Docker Machine.

### Virtual Machines

For local development, you should no longer worry about Virtual Machines at a project level as they are replaced by a
single Docker Machine instance. In environments where containers are meant to run on a cluster of hosts, orchestration
tools like [Kubernetes](http://kubernetes.io/) or [Swarm](https://www.docker.com/products/docker-swarm/) may be used
to distribute and coordinate containers across multiple Docker Hosts.

### Puppet / Ansible / configuration management

Goodbye, for local environments at least. Docker uses a conceptually different approach to configuration management than
Puppet. It’s perfectly possible to use tools like Puppet and Ansible from within a Dockerfile to get a container image
prepared though typically simple shell commands are preferred. The idea is that systems like Puppet are no longer needed
to manage upgrades across working servers/containers. When there are updates needed you will update the image, pull down
the new version of the image to your server and then stop the containers running the old version of the image and start
containers based on the new version of the image.
