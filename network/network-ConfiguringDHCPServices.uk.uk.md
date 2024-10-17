<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Configuring DHCP Services

The Dynamic Host Configuration Protocol \(DHCP\) enables client systems to obtain network configuration information from a DHCP server each time they connect to the network. The DHCP server is configured with a range of IP addresses and other network configuration parameters that clients need.

When you configure an Enterprise Linux system as a DHCP client, the client daemon, `dhclient`, contacts the DHCP server to obtain the networking parameters. As DHCP is broadcast-based, the client must be on the same subnet as either a server or a relay agent. If a client can't be on the same subnet as the server, a DHCP relay agent can be used to pass DHCP messages between subnets.

The server provides a lease for the IP address that it assigns to a client. The client can request specific terms for the lease, such as the duration. You can configure a DHCP server to limit the terms that it can grant for a lease. If a client remains connected to the network, `dhclient` automatically renews the lease before it expires. You can configure the DHCP server to provide the same IP address to a client, based on the MAC address of its network interface.

The advantages of using DHCP include the following:

- Centralized management of IP addresses

- Ease of adding new clients to a network

- Reuse of IP addresses reducing the total number of IP addresses that are required

- Reconfiguration of the IP address space on the DHCP server without needing to reconfigure each client

For more information about DHCP, see [RFC 2131](https://datatracker.ietf.org/doc/html/rfc2131). Likewise, see the following manual pages:

- `dhcpd(8)`
- `dhcp-options(5)`

## Setting Up the Server's Network Interfaces

By default, the `dhcpd` service processes requests on those network interfaces that connect them to subnets that are defined in the DHCP configuration file.

Suppose that a DHCP server has mutliple interfaces. Through its interface `enp0s1`, the server is connected to the same subnet as the clients that the server is configured to serve. In this case, `enp0s1` must be set in the DHCP service to enable the server to monitor and process incoming requests on that interface.

Before proceeding to either of the following procedures, ensure that you meet the following requirements:

- You have the proper administrative privileges to configure DHCP.
- You have installed the `dhcp-server` package.

  If not, install the package with the following command:

  ```
  sudo dnf install dhcp-server
  ```

Configure the network interfaces as follows:

- For IPv4 networks:
  1. Copy the `/usr/lib/systemd/system/dhcpd.service` file to the `/etc/systemd/system/` directory.

     ```
     sudo cp /usr/lib/systemd/system/dhcpd.service /etc/systemd/system/
     ```

  2. Edit the `/etc/systemd/system/dhcpd.service` by locating the line that defines the `ExecStart` parameter.

  3. Append the interface names on which the `dhcpd` service should listen.

     See the sample entries in bold.

     ```
     ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid $DHCPDARGS **_int1-name_** **_int2-name_**
     ```

  4. Reload the `systemd` manager configuration.

     ```
     sudo systemctl daemon-reload
     ```

  5. Restart the `dhcpd` service configuration.

     ```
     sudo systemctl restart dhcpd.service
     ```

     Alternatively, you can also type:

     ```
     sudo systemctl restart dhcpd
     ```

- For IPv6 networks:
  1. Copy the `/usr/lib/systemd/system/dhcpd6.service` file to the `/etc/systemd/system/` directory.

     ```
     sudo cp /usr/lib/systemd/system/dhcpd6.service /etc/systemd/system/
     ```

  2. Edit the `/etc/systemd/system/dhcpd6.service` file by locating the line that defines the `ExecStart` parameter.

  3. Append the names of the interfaces on which the `dhcpd6` service should listen.

     See the sample entries in bold.

     ```
     ExecStart=/usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd6.conf -user dhcpd -group dhcpd --no-pid $DHCPDARGS **_int1-name_** **_int2-name_**
     ```

  4. Reload the `systemd` manager configuration.

     ```
     sudo systemctl daemon-reload
     ```

  5. Restart the `dhcpd` service configuration.

     ```
     sudo systemctl restart dhcpd6.service
     ```

     Alternatively, you can also type:

     ```
     sudo systemctl restart dhcpd6
     ```

## Understanding DHCP Declarations

The way the DHCP provides services to its clients is defined through parameters and declarations in the `/etc/dhcp/dhcpd.conf` file for IPv4 networks and `/etc/dhcp/dhcpd6.conf` file for IPv6 networks. The file would contain details such as client networks, address leases, IP address pools, and so on.

**Note:** In a newly installed Enterprise Linux system, both the `dhcpd.conf` and `dhcpd6.conf` files are empty. If the server is being configured for DHCP for the first time, then you can use the templates so you can be guided in configuring the files. Type one of the following commands:

- For IPv4

  ```
  cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
  ```

- For IPv6

  ```
  cp /usr/share/doc/dhcp-server/dhcpd6.conf.example /etc/dhcp/dhcpd6.conf
  ```

Then when you open either file, examples, and explanations are available for reference.

The information in the configuration file consists of a combination of the following declarations:

- [Global Settings](network-ConfiguringDHCPServices.md#section_lhj_gwj_ptb)
- [Subnet Declarations](network-ConfiguringDHCPServices.md#section_gdx_vrk_ptb)
- [Shared-network Declarations](network-ConfiguringDHCPServices.md#section_uqm_wrk_ptb)
- [Host Declarations](network-ConfiguringDHCPServices.md#section_mn3_xrk_ptb)
- [Group Declarations](network-ConfiguringDHCPServices.md#section_hb5_xrk_ptb)

### Global Settings

Global parameters define settings that apply to all networks that are supported or serviced by the DHCP server.

Consider the following settings that would globally apply through out the entire network:

- Domain name of the company network: `example.com`.d
- Network's DNS servers: `dn1.example.com` and `dns2.example.com`
- Lease time assigned to all clients: 12 hours \(43200 seconds\)
- Maximum lease time that can be assigned: 24 hours \(86400 seconds\)

In this case, you would configure the global settings in the configuration file as follows:

```
option domain-name "example.com";
default-lease-time 43200;
max-lease-time 86400;

authoritative;
```

The `authoritative` parameter identifies the server as an official or primary server for DHCP services. The parameter is typically used in a setup that has multiple DHCP servers. Servers with the `authoritative` parameter have priority to process requests over servers without the parameter.

### Subnet Declarations

A `subnet` declaration provides details about a subnet to which the DHCP server is directly connected and where the systems in that subnet are also being served as clients.

Consider the following configuration of a DHCP server:

- The server's `enp0s1` interface is directly connected to the 192.0.2.0/24 network.
- The systems in the 192.0.2.0/24 network are DHCP clients.
- The topology of this client subnet is as follows:
  - Subnet's DNS server: 192.0.2.1.
  - Subnet gateway: 192.0.2.1.
  - Broadcast address: 192.0.2.255.
  - Address range for clients: 192.0.2.10 through 192.0.2.100.
  - Maximum lease time for each client: 86,400 seconds \(1 day\).

In this case, you would enter the following declaration in `dhcp.conf`:

```

subnet 192.0.2.0 netmask 255.255.255.0 {
  range 192.0.2.10 192.0.2.100;
  option domain-name-servers 192.0.2.1;
  option routers 192.0.2.1;
  option broadcast-address 192.0.2.255;
  max-lease-time 86400;
}
```

On an IPv6 network environment, a `subnet` declaration in the `dhcpd6.conf` file would resemble the following example:

```
subnet6 2001:db8:0:1::/64 {
  range6 2001:db8:0:1::20 2001:db8:0:1::100;
  option dhcp6.name-servers 2001:db8:0:1::1;
  max-lease-time 172800;
}
```

### Shared-network Declarations

You define a `shared-network` declaration if the DHCP server needs to provide servers to clients in other subnets that aren't directly connected to the server.

Consider the following example, which expands but slightly differs from the scenario in the preceding section:

- The DHCP server belongs to the 192.0.2.0/24 network but doesn't provide services to the systems in this network.
- The server processes requests from clients in the following remote subnets:
  - 192.168.5.0/24.
  - 198.51.100.0/24.
- The remote subnets share the same DNS server, but each subnet has its own router and IP address range.

In this case, you would enter the following declarations in `dhcp.conf`:

```
shared-network example {
 
  option domain-name-servers 192.168.2.1;
  ...

  subnet 192.168.5.0 netmask 255.255.255.0 {
    range 192.168.5.10 192.168.5.100;
    option routers 192.168.5.1;
  }

  subnet 198.51.100.0 netmask 255.255.255.0 {
    range 198.51.100.10 198.51.100.100;
    option routers 198.51.100.1;
  }
  ...
}

subnet 192.0.2.0 netmask 255.255.255.0 {
}

```

In the preceding example, the final `subnet` declaration refers to the server's own network and is outside the `shared-network` scope. The declaration is called an empty declaration because it defines the server's subnet. Because the server doesn't provide services to this subnet, no added entries are added, such as lease, address range, DNS information, and so on. Though empty, the declaration is required, otherwise, the `dhcpd` service doesn't start.

On an IPv6 network environment, a `shared-network` declaration in the `dhcpd6.conf` file would resemble the following example:

```
shared-network example {
  option domain-name-servers 2001:db8:0:1::1:1
  ...

  subnet6 2001:db8:0:1::1:0/120 {
    range6 2001:db8:0:1::1:20 2001:db8:0:1::1:100
  }

  subnet6 2001:db8:0:1::2:0/120 {
    range6 2001:db8:0:1::2:20 2001:db8:0:1::2:100
  }
  ...
}

subnet6 2001:db8:0:1::50:0/120 {
}
```

### Host Declarations

You define a `host` declaration if a client needs to have a static IP address.

Consider the following example of a client printer in the server's 192.0.2.0/24 network. This time, the server provides DHCP services to the subnet.

- Printer's MAC address: 52:54:00:72:2f:6e.
- Printer's IP address: 192.0.2.130

  **Important:** A client's fixed IP address must be outside the pool of dynamic IP addresses distributed to other clients. Otherwise, address conflicts might occur.

In this case, you would enter the following declaration in `dhcp.conf`:

```
host printer.example.com {
	hardware ethernet 52:54:00:72:2f:6e;
	fixed-address 192.0.2.130; 
}
```

Systems are identified by the hardware ethernet address, and not the name in the `host` declaration. Thus, the host name might change, but the client continues to receive services through the ethernet address.

On an IPv6 network environment, a `host` declaration in the `dhcpd6.conf` file would resemble the following example:

```
host server.example.com {
	hardware ethernet 52:54:00:72:2f:6e;
	fixed-address6 2001:db8:0:1::200;
}
```

### Group Declarations

You define a `group` declaration to apply the same parameters to multiple shared networks, subnets, and hosts all at the same time.

Consider this example:

- The DHCP server belongs to and serves the subnet 192.0.2.0/24.
- One client requires a fixed address, while the rest of the clients use dynamic IP addresses from the server.
- All the clients use the same DNS server.

In this case, you would enter the following declaration in `dhcp.conf`:

```

group {
  option domain-name-servers 192.0.2.1;

  host server1.example.com {
    hardware ethernet 52:54:00:72:2f:6e;
    fixed-address 192.0.2.130;
  }

  subnet 192.0.2.0 netmask 255.255.255.0 {
  range 192.0.2.10 192.0.2.100;
  option routers 192.0.2.1;
  option broadcast-address 192.0.2.255;
  max-lease-time 86400;
  }
}
```

On an IPv6 network environment, a `group` declaration in the `dhcpd6.conf` file would resemble the following example:

```
group {
  option dhcp6.domain-search "example.com";

  host server1.example.com {
    hardware ethernet 52:54:00:72:2f:6e;
    fixed-address 2001:db8:0:1::200;
  }

  host server2.example.com {
    hardware ethernet 52:54:00:1b:f3:cf;
    fixed-address 2001:db8:0:1::ba3;
  }
}

subnet6 2001:db8:0:1::/64 {
  range6 2001:db8:0:1::20 2001:db8:0:1::100;
  option dhcp6.name-servers 2001:db8:0:1::1;
  max-lease-time 172800;
}
```

## Activating the DHCP Services

All the DHCP services are defined in the server's `/etc/dhcp/dhcpd.conf` or `/etc/dhcp/dhcpd6.conf` file. To configure and then activate the configured services, follow these steps:

- For IPv4 networks:
  1. Open the `/etc/dhcp/dhcpd.conf` file.

  2. Add parameters and declarations to the file.

     For guidance, see [Understanding DHCP Declarations](network-ConfiguringDHCPServices.md#) or to the comments and notes in the `/usr/share/doc/dhcp-server/dhcpd.conf.example` template.

  3. Optionally, set the `dhcpd` service to start automatically in a server reboot.

     ```
     sudo systemctl enable dhcpd
     ```

  4. Start or restart the `dhcpd` service.

     ```
     sudo systemctl start dhcpd
     ```

- For IPv6 networks:
  1. Open the `/etc/dhcp/dhcpd6.conf` file.

  2. Add parameters and declarations to the file.

     For guidance, see [Understanding DHCP Declarations](network-ConfiguringDHCPServices.md#) or to the comments and notes in the `/usr/share/doc/dhcp-server/dhcpd6.conf.example` template.

  3. Optionally, set the `dhcpd6` service to start automatically in case of a server reboot.

     ```
     sudo systemctl enable dhcpd6
     ```

  4. Start or restart the `dhcpd` service.

     ```
     sudo systemctl start dhcpd6
     ```

## Recovering From a Corrupted Lease Database

The `dhcpd` service maintains lease information, such as IP addresses, MAC addresses, and lease expiry times, in the following flat-file databases:

- For DHCPv4: `/var/lib/dhcpd/dhcpd.leases`.
- For DHCPv6: `/var/lib/dhcpd/dhcpd6.leases`.

To prevent the lease database files from becoming too large with stale data, the `dhcpd` service periodically regenerates the files through the following mechanism:

1. The service renames the existing lease files:
   - `/var/lib/dhcpd/dhcpd.leases` is renamed to `/var/lib/dhcpd/dhcpd.leases~`
   - `/var/lib/dhcpd/dhcpd6.leases` is renamed to `/var/lib/dhcpd/dhcpd6.leases~`
2. The service re-creates brand new `dhcpd.leases` and `dhcpd6.leases` files.

If a lease database file is corrupted, you need to restore the lease database from the last known backup of the database.

Typically, the most recent backup of a lease database is the `*filename*.leases~` file.

**Note:** A backup instance is a snapshot taken at a particular point in time, and therefore might not reflect the latest state of the system.

Ensure that you have the required administrative privileges and complete the following steps:

- For DHCPv4
  1. Stop the `dhcpd` service:

     ```
     sudo systemctl stop dhcpd
     ```

  2. Rename the corrupt lease database:

     ```
     sudo mv /var/lib/dhcpd/dhcpd.leases /var/lib/dhcpd/dhcpd.leases.corrupt
     ```

  3. Restore the lease database from its corresponding `*filename*.leases~` backup file.

     ```
     sudo cp -p /var/lib/dhcpd/dhcpd.leases~ /var/lib/dhcpd/dhcpd.leases
     ```

  4. Start the `dhcpd` service:

     ```
     sudo systemctl start dhcpd
     ```

- For DHCPv6
  1. Stop the `dhcpd` service:

     ```
     sudo systemctl stop dhcpd6
     ```

  2. Rename the corrupt lease database:

     ```
     sudo mv /var/lib/dhcpd/dhcpd6.leases /var/lib/dhcpd/dhcpd6.leases.corrupt
     ```

  3. Restore the lease database from its corresponding `*filename*.leases~` backup file.

     ```
     sudo cp -p /var/lib/dhcpd/dhcpd6.leases~ /var/lib/dhcpd/dhcpd6.leases
     ```

  4. Start the `dhcpd6` service:

     ```
     sudo systemctl start dhcpd6
     ```
