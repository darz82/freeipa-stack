# FreeIPA docker-compose deployment with macvlan 802.1q bridged network

I wanted to run this in docker even though it really feels like it ought to be in its own vm.  
My infrastructure relies on one server in particular--the storage server.  VMs are all stored
on it, and if it doesn't come up, nothing comes up.  I decided to host my directory services
there for that reason.  It's a very simple bare ubuntu 20.04 host managing zfs attached storage,
iSCSI, and other file serving tasks.

It wants a lot of ports exposed:

	1. You must make sure these network ports are open:
		TCP Ports:
		  * 80, 443: HTTP/HTTPS
		  * 389, 636: LDAP/LDAPS
		  * 88, 464: kerberos
		UDP Ports:
		  * 88, 464: kerberos
		  * 123: ntp

I don't want to map all those ports onto my storage host's ip, so I came up with this.  It creates
a default network for the compose stack using the macvlan driver, which allows you to bind the 
network to a host interface or 802.1q subinterface.  In my case, the host has an LACP link 
aggregation group, bond0, and I'm binding the network to bond0.1502, which automatically creates
a subinterface on VLAN 1502, which puts any docker hosts in the stack on their own logical network
segment.

IPv6 support is required for freeipa to install; it specifically checks to make sure that `lo` has
the ::1 localhost address.  `enable_ipv6` alone is not enough, the default network being created 
wants an ipv6 subnet when ipv6 is enabled or it will complain.  This one is mine, [go get your own](https://simpledns.plus/private-ipv6).

If you run this compose stack as is, freeipa will come up in interactive server install mode, and 
becase compose is combining logs from all containers, it is not interactive.  Placing `ipa-server-install-options`
on the data volume with appropriate values will give you unattended install mode.
