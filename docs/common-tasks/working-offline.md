# Working Offline

A common desire is the ability to leverage the full development environment provided to work
offline. When attempting this users are often thwarted trying to access containers which
appear to be successfully running.

This issue is due to a failure to resolve domain names to the appropriate IP address for the
container. This affects those using OS X and appears to be the result of a failure of the name
resolution system to operate when no network interface appears connected. Typical error messages
in browsers warn of being disconnected from the internet. Specifically, Chrome will include the 
error code ERR_INTERNET_DISCONNECTED. In these instances, a test command like 
`curl -v http://www.project.vm/` succeeds as does attempting to use the IP address of a container
within a browser. The command `rig dns-records` will display a mapping between all known
containers and IP addresses. The `docker inspect CONTAINER_NAME` command can be used to view more
detailed information about a specific container including its IP address and should be used on
systems not supporting the `rig` application.

## Offline DNS Workaround

The work around for this issue is to use the `rig dns-records` command and copy the output
to your `/etc/hosts` file. You'll need to update this file any time you start or stop project
containers and you should clean entries from it when you reconnect to a network. On systems
which do not support the `rig` application the `docker inspect` command can be used to
build a mapping of IP addresses to domain names.

!!! warning "Network Changes Can Require Restart"
    Network changes such as connecting or disconnecting an interface or VPN can require a restart
    of your Outrigger environment via `rig restart` or a re-execution of the `rig dns`
    command to ensure routing of traffic to containers and DNS resolution is properly configured.

## Additional Information

Those seeking additional information about the root cause of this issue and wishing to explore
potential solutions can read further information at the following URLs. Note that none of the
purported solutions other than that described above has proven successful in testing. Note that
some of these links refer to DNS utilities used in older OS X versions.

* [http://serverfault.com/questions/22419/set-dns-server-on-os-x-even-when-without-internet-connection](http://serverfault.com/questions/22419/set-dns-server-on-os-x-even-when-without-internet-connection)
* [http://apple.stackexchange.com/questions/202887/mac-doesnt-use-local-dns-for-local](http://apple.stackexchange.com/questions/202887/mac-doesnt-use-local-dns-for-local)
* [http://superuser.com/questions/418833/using-dnsmasq-on-os-x-when-not-connected-to-the-internet](http://superuser.com/questions/418833/using-dnsmasq-on-os-x-when-not-connected-to-the-internet)
* [http://apple.stackexchange.com/questions/26616/dns-not-resolving-on-mac-os-x](http://apple.stackexchange.com/questions/26616/dns-not-resolving-on-mac-os-x)

This same issue affects other projects as well.

* [https://github.com/basecamp/pow/issues/471](https://github.com/basecamp/pow/issues/471)
* [https://github.com/basecamp/pow/issues/104](https://github.com/basecamp/pow/issues/104)

A theoretical solution to this issue would be to have a network interface that always appeared
active triggering the attempt to resolve the domain name. The following post discusses creation
of a virtual interface to satisfy this requirement. The mechanism mentioned by bmasterswizzle and
Alex Gray did not prove effective in testing. The interface could be created successfully but
never brought up to a state where OS X considered it active.

* [http://stackoverflow.com/questions/87442/virtual-network-interface-in-mac-os-x](http://stackoverflow.com/questions/87442/virtual-network-interface-in-mac-os-x)

Some additional information and demonstrations of utilities for diagnostics can be found at

* [http://apple.stackexchange.com/questions/26616/dns-not-resolving-on-mac-os-x](http://apple.stackexchange.com/questions/26616/dns-not-resolving-on-mac-os-x)
* [http://serverfault.com/questions/478534/how-is-dns-lookup-configured-for-osx-mountain-lion](http://serverfault.com/questions/478534/how-is-dns-lookup-configured-for-osx-mountain-lion)
