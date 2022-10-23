# dhclient-pd
`dhclient-pd` is a Python 3 script that takes an incoming IPv6 prefix delegation
from dhclient and assigns subnet prefix addresses to one or more additional
interfaces on the system. The default behaviour is to look for
`iface xxx inet6 manual` lines in `/etc/network/interfaces` and automatically
assign prefix addresses to those interfaces. However, if you are not on a
Debian system you can provide a custom list of interfaces.

## Why two scripts?
dhclient actually sources the hook scripts, which means that the hook script
must be valid shell script code. However, working with IP addresses in shell
scripting is both error prone and complex, so I implemented that logic in
python instead. Python was chosen because it is nearly always available on any
modern linux system.

# Requirements
* a modern version of python3

# Installation

    sudo cp ./src/prefix-delegation /etc/dhcp/dhclient-exit-hooks.d/prefix-delegation
    sudo chmod 644 /etc/dhcp/dhclient-exit-hooks.d/prefix-delegation
    sudo cp ./src/dhclient-pd /usr/local/bin/dhclient-pd
    sudo chmod 755 ./src/dhclient-pd /usr/local/bin/dhclient-pd

# Configuration
## Debian
Edit your `/etc/network/interfaces` file and add `iface xxx inet6 manual`
statements for all interfaces that you want to automatically assign IPv6 prefix
addresses to, e.g.:

    iface eth1.3 inet6 manual

If you have some interfaces configured with `inet6 manual` that you do not want
to assign prefixes to, edit the `prefix-delegation` hook script and add one or
more `-x INTERFACE` arguments to the dhclient-pd command to exclude those.

    /usr/local/bin/dhclient-pd -x eth1.3 -x eth1.4

## Custom
Edit the `prefix-delegation` hook script and add one or more `-i INTERFACE`
arguments to the dhclient-pd command. `-i/--interface` is mutually exclusive
to `-x/--exclude-interface` as the interfaces file detection logic is
disabled if a manual list of interfaces is provided.

    /usr/local/bin/dhclient-pd -i eth1.3 -i eth1.4
