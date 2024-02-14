## Introduction

`pf`, short for *packet filter*, is a commonly used firewall on BSD systems.

The firewall controls network traffic based on the rules in the configuration file. For each incoming and outgoing packet, the firewall evaluates the filtering rules sequentially (from top to bottom). By default, each packet passes through unless a specific rule in the firewall blocks it. Packets which are part of an existing connection pass through without being evaluated against the rules (`pf` is a stateful firewall).

A rule can apply (match) to none, some, or all of the packets. **The last matching rule decides whether to allow or block the packet.**

### Compatibility

This guide has been tested on **FreeBSD 13.0-RELEASE**. It should be compatible with all recent FreeBSD and OpenBSD versions.

## Start and Stop

FreeBSD and OpenBSD both come with `pf` pre-installed. On OpenBSD, `pf` is enabled by default. On FreeBSD, you have to enable it before using it.

### Enable and Start

#### Create the Configuration File

By default, `pf` loads the configuration in the file `/etc/pf.conf`. Before starting `pf`, create this file and add to it a single rule:

    pass log all

Save the file.

#### Enable pf

To enable `pf`, add an entry in the `/etc/rc.conf` file using the `sysrc` command:

    # sysrc pf_enable="YES"

Check if the `pf_enable` entry has been added:

    # sysrc pf_enable

#### Start the pf Service (First Time)

After enabling `pf` in `rc.conf`, reboot the system, or start the `pf` service manually:

    # service pf start

This will start `pf` and load the default configuration file `/etc/pf.conf`.

Check if the `pf` service is enabled:

    # service pf status

#### Start pf

`pfctl` is the tool used for managing `pf`.

Check if `pf` is running:

    # pfctl -s Running

Start `pf` and load the default configuration file `/etc/pf.conf`:

    # pfctl -e

Start `pf` and load a specific configuration file:

    # pfctl -e -f /path/to/config_file

### Stop and Disable

#### Stop pf

Stop the firewall:

    # pfctl -d

You can also stop the `pf` service :

    # service pf stop

#### Disable pf

To disable `pf`, update the `pf_enable` entry in the `rc.conf` file using the `sysrc` command:

    # sysrc pf_enable="NO"

## View Current Ruleset

Use `pfctl` with the `-s` option and the modifier `rules` to view the current filter rules:

    # pfctl -s rules

To view the current NAT redirection rules, use the modifier `nat`:

    # pfctl -s nat

See a complete overview (excluding the list of tables) of the current setup:

    pfctl -s all

View summary information about the overall activity of the firewall:

    # pfctl -s info

## Loading Rulesets

### Load New Ruleset

Load a new configuration when `pf` is already running:

    # pfctl -f /path/to/config_file

**Note:** Do this every time you make changes to the default configuration file `/etc/pf.conf`.

### Dry-runs

Do a dry-run to check if the configuration is valid:

    # pfctl -vnf /path/to_config_file

Errors are shown at the prompt. If the dry-run was successful, it shows the complete configuration.

### Temporarily Loading Rulesets

#### Timer Script

Use a shell script with a timer to temporarily load a new configuration file. When the timer expires, the script rolls back the new configuration and reloads the old configuration.

The examples in this section assume that `pf` is already running with the default configuration file `/etc/pf.conf`.

Create a new file titled `pftimer.sh` in your home directory `/home/your_user_name`. Add to it the lines below:

    #!/bin/sh

    # load the new configuration
    pfctl -f /etc/pf.conf.NEW

    # set a timer (in seconds) - as much as you need
    sleep 30

    # bring back the old working configuration 
    pfctl -f /etc/pf.conf 

Save the file. Change the ownership and permissions of this script such that your user account can write to it, but only root can execute it.

    # chown root /home/your_user_name/pftimer.sh

    # chmod 766 /home/your_user_name/pftimer.sh

#### Temporarily Block All Traffic

Create a new configuration file `/etc/pf.conf.NEW`. Add to it a single line:

    block all

Save the file. The above rule blocks all (incoming and outgoing) traffic.

Temporarily enable the new configuration using the timer script:

    # /home/your_user_name/pftimer.sh

**Note:** If you lose access to your own server because of a misconfiguration, use the [Vultr Web Console](https://www.vultr.com/docs/vultr-web-console-faq/) to log back in and fix the settings.

#### Temporarily Allow All Traffic

Create a new configuration file `/etc/pf.conf.NEW` with a single rule:

    pass all

Save the file and enable it with the timer script to allow all (incoming and outgoing) traffic to pass through temporarily.

## Allow Selected Traffic

### Allow Only Outgoing Traffic

The simplest configuration is to block all incoming traffic and allow all outgoing traffic:

    block in all
    pass out all

With this as a starting point, add rules for the incoming traffic you want to allow.

### Allow SSH Traffic

To allow SSH connections, allow incoming TCP and UDP traffic over the `ssh` port:

    pass in proto { tcp udp } to port ssh

### Allow HTTP(S) Traffic

HTTP and HTTPS traffic go over the TCP protocol. To allow incoming HTTP and HTTPS traffic, add these lines in the configuration file:

    pass in proto tcp to port http
    pass in proto tcp to port https

Or, write them together:

    pass in proto tcp to port { http https }

### Allow Specific IPs

To allow a specific IP address (for example, 203.0.113.1) to access the server, add this line to the configuration file:

    pass quick from 203.0.113.1

**Note:** Write rules containing the `quick` keyword at the beginning of the filtering rules.

### Allow Certbot

[Certbot needs to communicate with the Let's Encrypt servers via HTTP(S) traffic over the standard ports. For challenges, allow outgoing TCP and UDP traffic on port 53](https://letsencrypt.org/docs/integration-guide/#firewall-configuration).

    pass out proto { tcp udp } to port { 53 80 443 }

The above rule allows outgoing TCP and UDP traffic on the HTTP and HTTPS ports, as well as on port 53. You may take a more casual approach and allow all outgoing TCP and UDP traffic to pass through:

    pass out proto { tcp udp }

## Multiple Interfaces

When a system has multiple interfaces, each interface needs separate filtering rules. It is common to filter packets coming in on the external interface but have free movement on all internal interfaces. You might want to redirect certain types of packets to certain interfaces.

The examples in this section are based on the following setup consisting of a Virtual Machine (VM) on a Virtual Private Cloud (VPC):

* The external (public facing) interface, `vtnet0`, receives Internet traffic.

* `lo0` is the localhost interface.

* `lo1` is the (virtual internal) interface of the VM.

* A web-server listening on `lo1` handles HTTP(S) traffic.

* There is free movement on internal interfaces while the public interface is protected.

Adapt the rules and interface names in the examples below based on your own configuration.

Check the list of interfaces available for `pf` to control:

    # pfctl -s Interfaces

### Allow All Traffic on Internal Interfaces

    pass quick on { lo0 lo1 }

This rule allows traffic on the internal interfaces to pass freely.

### Port Forwarding (Redirection)

Forwarding rules redirect requests to different ports and interfaces.

    rdr on vtnet0 proto tcp to port http -> lo1 port http
    rdr on vtnet0 proto tcp to port https -> lo1 port https

The above rules redirect incoming TCP requests on the HTTP and HTTPS ports on the external interface to the corresponding ports on the internal interface `lo1`.

**Note:** In the configuration file, redirection rules and NAT rules must be written before filtering rules (`pass` and `block`).

### Network Address Translation (NAT)

In the outgoing response from `lo1`, the source address is the internal network address. But in the response received by the client, the source address needs to be the server's public IP address. NAT rules handle this.

    nat on vtnet0 from lo1 to any -> vtnet0

This rule acts on the `vtnet0` interface on packets coming from `lo1` and going out to any destination. It translates the packet's source address from the (internal) address of `lo1` to the (external) address of `vtnet0`.

### Allow Incoming TCP Traffic

Allow incoming TCP traffic to pass through on the external interface.

    pass in on vtnet0 proto tcp to lo1 port { http https }

This rule allows incoming TCP packets to the HTTP(S) port to pass through. The earlier `rdr` rules have already redirected the packets to the `lo1` interface.

## Logging

Enable logging on `pf`:

    # sysrc pflog_enable="YES"

Reboot the system, or start the logging service manually:

    # service pflog start

The logging service uses the `pflog0` interface. The default log file is `/var/log/pflog`. Logs are written in binary format.

**Note:** By default, no logs are produced. Specify the keyword `log` in a rule to log traffic matching that rule.

View the logs in real time using `tcpdump` with the `-i` option:

    # tcpdump -ni pflog0

Use the `-r` option to view log files:

    # tcpdump -nr /var/log/pflog

The `-n` option in the above commands is to show raw IP addresses.

## References

The standard working reference is the [FreeBSD manual page](https://www.freebsd.org/cgi/man.cgi?query=pf.conf). The FreeBSD Handbook has a [chapter on Firewalls](https://docs.freebsd.org/en/books/handbook/firewalls/) with explanations and examples.
