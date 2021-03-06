# ocat(1) - OnionCat creates a transparent IPv6 layer on top of Tor's hidden services.

ocat, 2019-08-19
  
gcat - GarliCat is like OnionCat but it works with I2P instead of Tor.

# Synopsis

```
ocat -i onion_id                      (1st form)\fP
ocat -o IPv6_address                  (2nd form)\fP
ocat [OPTION\fP] onion_id                (3rd form)\fP
ocat -R [OPTION\fP]                      (4th form)\fP
gcat [OPTION\fP] i2p_id                  (5th form)\fP

```

# Description

OnionCat creates a transparent IPv6 layer on top of Tor's hidden services or
I2P's tunnels. It transmits any kind of IP-based data transparently through the
Tor/I2P network on a location hidden basis. You can think of it as a
peer-to-peer VPN between hidden services.

OnionCat is a stand-alone application which runs in userland and is a connector
between Tor/I2P and the local OS. Any protocol which is based on IP can be
transmitted. Of course, UDP and TCP (and probably ICMP) are the most important
ones but all other protocols can also be forwarded through it.

OnionCat opens a TUN device and assigns an IPv6 address to it. All packets
forwarded to the TUN device by the kernel are forwarded by OnionCat to other
OnionCats listening on Tor's hidden service ports or I2P's server tunnels. The
IPv6 address depends on the _onion_id_ or the i2p_id, respectively. The
_onion_id_ is the hostname of the locally configured hidden service (see
**tor(8)**). Depending on the configuration of Tor the _onion_id_ usually
can be found at _/var/lib/tor/hidden_service/hostname_ or similar location.
The _i2p_id_ is the 80 bit long Base32 encoded hostname of the I2P server
tunnel.


## OPTIONS


* **-4**  
  Enable IPv4 forwarding. See http://www.cypherpunk.at/onioncat/wiki/IPv4 for further
  information on IPv4.  
  Native IPv4 forwarding is deprecated. The recommended solution for IPv4
  forwarding is to build a IPv4-through-IPv6 tunnel through OnionCat.
* **-5** _'socks5'|'direct'_  
  This option has a mandatory argument which is literally either _socks5_ or
  _direct_.  
  By default OnionCat uses SOCKS4A (version 4a) to connect to the anonymization
  network proxy (e.g. Tor). With this option set to _socks5_, OnionCat uses
  SOCKS5 (version 5 as specified in RFC1928), currently with no authentication
  mechanism. As of today it actually makes no difference but it might be
  desireable in future.  
  If _direct_ is used, OnionCat does not connect through to SOCKS server but
  instead it connects directly to the remote peers using the hosts lookup
  mechanism (see option **-H**). This feature is experimental and turns
  OnionCat into a virtual switch based on regular Internet transport instead of
  Tor.
* **-a**  
  OnionCat creates a log file at $HOME/.ocat/connect_log. All incoming connects are
  logged to that file. $HOME is determined from the user under which OnionCat runs
  (see option -u).
* **-b**  
  Run OnionCat in background. This is default. OnionCat will detach from a running
  shell and close standard IO if no log file is given with option -L.
* **-B**  
  Run OnionCat in foreground. OnionCat will log to stderr by default.
* **-C**  
  Disable the local controller interface. The controller interfaces listens on
  localhost (127.0.0.1 and ::1 port 8066) for incoming connections. It's
  currently used for debugging purpose and not thread-safe and does not have any
  kind of authentication or authorization mechanism. Hence, it should not be used
  in production environments.
* **-d** _n_  
  Set debug level to _n_. Default = 7 which is maximum. Debug output will
  only be created if OnionCat was compiled with option DEBUG (i.e. configure was
  run with option --enable-debug).
* **-e** _ifup_  
  Execute script _ifup_ to bring up the tunnel interface.  
  OnionCat will create a new tunnel interface and execute _ifup_ immediatly
  after opening the network interface. This is intended as a universial interface
  for configuring the tunnel device and do additinal tasks when starting
  OnionCat.  The script is executed with the same privilege as OnionCat is
  started, i.e. before dropping privileges. This typically is root. The script is
  run only once at startup.
  
  See below in section EXAMPLES for a typical Linux ifup shell script.
  
  OnionCat executes the file _ifup_ with a call to execlp(3) and will pass
  the following environment variables: 
  
  **OCAT_IFNAME**  
  This variable contains the name of the network interface, e.g. "tun0".
   
  **OCAT_ADDRESS**  
  This variable contains the IPv6 address which is associated with this instance
  of OnionCat and its hidden service address.
  
  **OCAT_PREFIXLEN**  
  This variable contains the prefix length of the IPv6 prefix which typically is
  48.
  
  **OCAT_PREFIX**  
  This variable contains the IPv6 prefix, i.e. the network. This typically is
  fd87:d87e:eb43:: in Onioncat (Tor) mode and fd60:db4d:ddb5:: in Garlicat
  (I2P) mode.
  
* **-f** _config file_  
  Read initial configuration from _config file_. 
* **-g** _hosts_path_  
  Set the path to the hosts file. This option automatically enables option -H
  (see there). If -g is not set, the path defaults to the system hosts file
  which typically is /etc/hosts on Un*x systems.
* **-H**  
  This option enables the hosts reverse lookup. If OnionCat receives a packet to
  a destination IPv6 address within the OnionCat prefix, it translates it
  directly to a .onion hostname by default. If option -H is enabled, OnionCat
  instead looks up the hostname in the hosts file (see also -g). This option is
  always enabled when OnionCat is used in Garlicat mode for I2P and it is required
  with V3 hidden services of Tor (see below).
* **-h**  
  Display short usage message and shows options.
* **-i**  
  Convert _onion_id_ to IPv6 address and exit.
* **-I**  
  Run OnionCat in GarliCat mode. Using this option is identical to running OnionCat
  with the command name **gcat**.
* **-l** _[ip:]port_  
  Bind Onioncat to specific _ip _ and/or _port_ number for incoming
  connections. It defaults to 127.0.0.1:8060. This option could be set
  multiple times. IPv6 addresses must be given in square brackets.  
  The parameter _"none"_ deactivates the listener completely. This is for
  special purpose only and shall not be used in regular operation.
* **-L** _log_file_  
  Log output to _log_file_. If option is omitted, OnionCat logs to syslog if
  running in background or to stderr if running in foreground. If syslogging is
  desired while running in foreground, specify the special file name "syslog" as
  log file.
* **-o** _IPv6 address_  
  Convert _IPv6 address_ to _onion_id_ and exit program.
* **-p**  
  Use TAP device instead of TUN device. There are a view differences. See \fBTAP
  DEVICE\fP later.
* **-P** _[pid file]_  
  Create _pid file_ at _pid_file_. If the option parameter is omitted OC
  will create a pid file at **/var/run/ocat.pid**. In the latter case it MUST
  NOT be the last option in the list of options.
* **-r**  
  Run OnionCat as root and do not change user id (see option **-u**).
* **-R**  
  Use this option only if you really know what you do!  OnionCat generates a
  random local onion_id. With this option it is not necessary to add a hidden
  service to the Tor configuration file **torrc**.  One might use OnionCat
  services within Tor as usually but it is NOT possible to receive incoming
  connections. If you plan to also receive connections (e.g.  because you provide
  a service or you use software which opens sockets for incoming connections
  like Bitorrent) you MUST configure a hidden service and supply its hostname to
  OnionCat on the command line.
  Please note that this option does only work if the remote OC does not run in
  unidirectional mode which is default since SVN version 555 (see option
  **-U**).
* **-s** _port_  
  Set OnionCat's virtual hidden service port to _port_. This should usually
  not be changed.
* **-t** _(IP|[IP:]port)_  
  Set Tor SOCKS _IP_ and/or _port_. If no _IP_ is specified 127.0.0.1
  will be used, if no _port_ is specified 9050 will be used as defaults. IPv6
  addresses must be escaped by square brackets.  
  The special parameter _"none"_ disables OnionCat from making outbound
  connections. This shall be used only in special test scenarios.
* **-T** _tun_dev_  
  TUN device file to open for creation of TUN interface. It defaults to
  /dev/net/tun on Linux and /dev/tun0 on most other OSes, or /dev/tap0 if TAP
  mode is in use. Setup of a TUN device needs root permissions. OnionCat
  automatically changes userid after the TUN device is set up correctly.
* **-U**  
  Deactivate unidirectional mode. Before SVN version 555 OnionCat ran only in
  bidirectional mode. This is that a connection to another OC was used for
  outgoing _and_ incoming packets. Since this could be a security risk under
  certain conditions, unidirectional mode was implemented in SVN r555 and set to
  default. With this option bidirectional mode can be enabled again. Please note
  that this does not interoperate with option **-R** if the remote OC is
  working in unidirectional mode.
* **-u** _username_  
  _username_ under which ocat should run. The uid is changed as soon as possible
  after tun device setup. 
  

## TAP DEVICE

Usually OnionCat opens a TUN device which is a layer 3 interface. With option
**-p** OnionCat opens a TAP device instead which is a virtual ethernet
(layer 2) interface.


# Examples

A typical ifup script for OnionCat for a modern Linux distribution using the
\`ip\` command for configuring network related stuff could look like the
following:

.in +3n
    #!/bin/sh
    
    ip address add $OCAT_ADDRESS/$OCAT_PREFIXLEN dev $OCAT_IFNAME
    ip link set $OCAT_IFNAME up 


# Onioncat and V3 Hidden Services

Originially Tor's v2 hidden service addresses had a binary length of 80 bits.
This made it possible to let OnionCat map hidden service addresses to IPv6
addresses and vice versa. The development of OnionCat started in 2008, and this
held for a very long time until recently Tor came up with version 3 of hidden
services. To comply with ongoing development in the field of cryptography the
new hidden service addresses of Tor (since version 0.3.2) are much bigger,
meaning 336 bits. This obviously does not fit into an IPv6 address, hence,
Onioncat is not able any more to translate back and forth between IPv6 and v3
onion addresses.

As a solution OnionCat offers the possibility to do an external hostname lookup
within /etc/hosts instead. Please note that for security reasons, OnionCat
does not use the system resolver, it definitely just reads the local hosts
file. The big drawback for OnionCat is that with v3 hidden services OnionCat
does not work out of the box any more. It requires that the destionations are
configured manually beforehand.

To connect to a v3 hidden service, on the client side add a line to your
/etc/hosts with the IPv6 address and the v3 hostname and run OnionCat with
the additional option **-H**. The hosts entry could look like this (in one
line!):

**fd87:d87e:eb43:45g6:3bbb:9fxf:5877:4319 tulqpcvf7Oeuxzjod6odrpO77ryujc7o0g7kw6c76q9cbnbi7rqskxid.onion**

If this client also has a v3 hidden service, you have to enter its
IPv6/hostname pair to the hosts file on the opposite site as well, except you
use the option **-U** on the other side.

Please note that you could pick any IPv6 address in this case, although I
suggest to truncate the long hostname just to the last 16 characters for use
with OnionCat, e.g. truncate
"tulqpcvf7Oeuxzjod6odrpO77ryujc7o0g7kw6c76q9cbnbi7rqskxid.onion" to
"6q9cbnbi7rqskxid.onion" and use it as parameter for OnionCat.


# Notes

This man page is still not finished...


# Files

$HOME/.ocat/connect_log


# Author

Concepts, software, and man page written by Bernhard R. Fischer
&lt;[bf@abenteuerland.at](mailto:bf@abenteuerland.at)&gt;. Package maintenance and additional support by Ferdinand
Haselbacher, Daniel Haslinger &lt;[creo-ocat@blackmesa.at](mailto:creo-ocat@blackmesa.at)&gt;, and Wim Gaethofs.


# See Also

OnionCat project page https://www.onioncat.org/

OnionCat source packages are found at https://www.cypherpunk.at/ocat/download/Source/

Tor project homepage https://www.torproject.org/

I2P project homepage https://geti2p.net/


# Copyright

Copyright 2008-2019 Bernhard R. Fischer.

This file is part of OnionCat.

OnionCat is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, version 3 of the License.

OnionCat is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with OnionCat. If not, see &lt;http://www.gnu.org/licenses/&gt;.

