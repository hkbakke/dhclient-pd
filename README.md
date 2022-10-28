# dhclient-pd
`dhclient-pd` is a script that takes an incoming IPv6 prefix delegation from
`dhclient` and assigns subnet prefix addresses to one or more additional
interfaces on the system. It is also a standalone tool that may be called from
any source, not just `dhclient`. Only the hook script is `dhclient` specific.

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
    sudo mkdir /etc/dhclient-pd

# Configuration
Create `/etc/dhclient-pd/interfaces.json`. Example syntax:

    [
        {
            "name": "enp4s0f0.3",
            "prefix_id": "3"
        },
        {
            "name": "enp4s0f1.4",
            "prefix_id": "4"
        },
        {
            "name": "enp4s0f1.5",
            "prefix_id": "af1"
        },
        {
            "name": "enp4s0f1.6",
            "prefix_id": "6"
        },
        {
            "interface_ids": [
                "::1",
                "::3"
            ],
            "name": "lo",
            "prefix_id": "0"
            "prefix_length: 128
        }
    ]

`name` and `prefix_id` are mandatory. Note that `prefix_id` is a string
containing the hex representation of the id, so the value needs to be quoted to
avoid being interpreted as an integer if there are no letters in the id.
`interface_ids` is set to `["::1"]` by default unless overriden. You may define
multiple `interface_ids` within the same prefix, which is normally useful only
for loopback addresses (/128). If you want to have multiple different prefixes
on the same interface, just add another configuration block for that interface
with an unique `prefix_id`.

# Manual execution
You are not bound to dhclient. The hook script manages all the dhclient
specifics. `dhclient-pd` is a standalone tool that may be called from any
source. Typically you would call it manually with:

    dhclient-pd config -p $prefix [-o $old_prefix]

You must replace `$prefix` with your actual delegated prefix.

If you run `config` without any prefix options nothing will happen as
`dhclient-pd` then doesn't know which prefixes to manage.

To quickly show the configuration:

    dhclient-pd show
