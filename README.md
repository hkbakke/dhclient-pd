# dhclient-pd
`dhclient-pd` is a Python 3 script that takes an incoming IPv6 prefix delegation
from dhclient and assigns subnet prefix addresses to one or more additional
interfaces on the system. It also supports looback address assignments.

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
Edit the `prefix-delegation` hook script and add one or more `-i INTERFACE`
arguments to the dhclient-pd command. Shell style wildcard pattern matching is
supported.

    # match eth1 and all subinterfaces of eth2
    /usr/local/bin/dhclient-pd -i eth1 -i "eth2.*"

If your interface patterns also matches some interface that you do not want to
assign addresses to, e.g. your WAN interface, you can add one or more
`-x INTERFACE` arguments to the dhclient-pd command to exclude those. Shell
style wildcard pattern matching is supported.

    # match all subinterfaces starting with eth, but exclude eth2.1
    /usr/local/bin/dhclient-pd -i "eth*.*" -x "eth2.1"

There is also support for loopback addresses. These are all assigned from the
same `/64` subnet prefix, have addresses with a prefix length of `/128` and you
must provide the interface identifier, i.e. the non-prefix part of the address.
They are always assigned to the `lo` interface.

    # match all subinterfaces starting with eth, but exclude eth2.1. Also
    # configure two loopback addresses.
    /usr/local/bin/dhclient-pd -i "eth*.*" -x "eth2.1" -l "::1" -l "::1.1.1.1"

# Manual execution
If you want to troubleshoot or test you can execute the script manually by
providing the minimum set of environment variables. Set `new_ip6_prefix` to
your own delegated prefix.

    reason=REBIND6 new_ip6_prefix="ab12:cd34:ef56::/48" dhclient-pd -i "enp4s0f*.*" -x enp4s0f0.2 -l ::1 -l ::2
