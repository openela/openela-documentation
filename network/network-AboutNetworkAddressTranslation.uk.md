<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# About Network Address Translation

Network Address Translation \(NAT\) is a process that assigns a public address to a computer or a group of computers inside a private network by using a different address scheme. The public IP address masquerades all the requests as though they're going to one server, rather than several servers. NAT is useful for limiting the number of public IP addresses that an organization must finance. NAT also provides extra security by hiding the details of internal networks.

The `netfilter` kernel subsystem provides the `nat` table to implement NAT, in addition to its tables for packet filtering. The kernel consults the `nat` table whenever it handles a packet that creates a new incoming or outgoing connection.

By default, IP forwarding is enabled and the system can route packets among configured network interfaces. To check whether IP forwarding is enabled, use the following command:

```
sudo sysctl -a | grep ip_forward
```

In the ensuing output, a value of `1` for `net.ipv4.ip_forward` indicates that IP forwarding is enabled.

You can change the status of IP forwarding on the system by using the following command:

```
sudo sysctl -w net.ipv4.ip_forward=0|1
```

The new status is displayed when you run the command. To make the change persist across system reboots, copy the command output line and add it to the `/etc/sysctl.conf` file.

You can also use the Firewall Configuration GUI \(`firewall-config`\) to configure masquerading and port forwarding.

