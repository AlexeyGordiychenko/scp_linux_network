# Linux Network (Individual project)

Linux networks configuration on virtual machines.

## Task

### Part 1. **ipcalc** tool

#### Start a virtual machine (hereafter -- ws1). Define and write in the report:

  + network address of *192.167.38.54/13*
  + conversion of the mask *255.255.255.0* to prefix and binary, */15* to normal and binary, *11111111.11111111.11111111.11110000* to normal and prefix
  + minimum and maximum host in *12.167.38.4* network with masks: */8*, *11111111.11111111.00000000.00000000*, *255.255.254.0* and */4*
  + whether an application running on localhost can be accessed with the following IPs: *194.34.23.100*, *127.0.0.2*, *127.1.0.1*, *128.0.0.1*
  + which of the listed IPs can be used as public and which only as private: *10.0.0.45*, *134.43.0.2*, *192.168.4.2*, *172.20.250.4*, *172.0.2.1*, *192.172.0.1*, *172.68.0.2*, *172.16.255.255*, *10.10.10.10*, *192.169.168.1*
  + which of the listed gateway IP addresses are possible for *10.10.0.0/18* network: *10.0.0.1*, *10.10.0.2*, *10.10.10.10*, *10.10.100.1*, *10.10.1.255*

### Part 2. Static routing between two machines

#### Start two virtual machines (hereafter -- ws1 and ws2)

+ View existing network interfaces with the `ip a` command
+ Describe the network interface corresponding to the internal network on both machines and set the following addresses and masks: ws1 - *192.168.100.10*, mask */16 *, ws2 - *172.24.116.8*, mask */12*
+ Run the `netplan apply` command to restart the network service

#### 2.1 Adding a static route manually
+ Add a static route from one machine to another and back using a
`ip r add` command.
+ Ping the connection between the machines

#### 2.2 Adding a static route with saving
+ Restart the machines
+ Add static route from one machine to another using *etc/netplan/00-installer-config.yaml* file
+ Ping the connection between the machines

### Part 3. **iperf3** utility

In this task you need to use ws1 and ws2 from [Part 2](#part-2-static-routing-between-two-machines).

+ Convert and write results in the report: 8 Mbps to MB/s, 100 MB/s to Kbps, 1 Gbps to Mbps
+ Measure connection speed between ws1 and ws2

### Part 4. Network firewall

In this task you need to use ws1 and ws2 from [Part 2](#part-2-static-routing-between-two-machines).

#### 4.1 **iptables** utility

+ Create a */etc/firewall.sh* file simulating the firewall on ws1 and ws2:
  ```shell
  #!/bin/sh

  # Deleting all the rules in the "filter" table (default).
  iptables -F
  iptables -X
  ```
+ The following rules should be added to the file in a row:
  + on ws1 apply a strategy where a deny rule is written at the beginning and an allow rule is written at the end (this applies to points 4 and 5)
  + on ws2 apply a strategy where an allow rule is written at the beginning and a deny rule is written at the end (this applies to points 4 and 5)
  + open access on machines for port 22 (ssh) and port 80 (http)
  + reject *echo reply* (machine must not ping, i.e. there must be a lock on OUTPUT)
  + allow *echo reply* (machine must be pinged)
+ Run the files on both machines with `chmod +x /etc/firewall.sh` and `/etc/firewall.sh` commands.
  + Describe in the report the difference between the strategies used in the first and second files.

#### 4.2 **nmap** utility

+ Use **ping** command to find a machine which is not pinged, then use **nmap** utility to show that the machine host is up
*Check: nmap output should say: `Host is up`*.

### Part 5. Static network routing

Network: \
![part5_network](misc/images/part5_network.png)

#### Start five virtual machines (3 workstations (ws11, ws21, ws22) and 2 routers (r1, r2))

#### 5.1 Configuration of machine addresses
+ Set up the machine configurations in *etc/netplan/00-installer-config.yaml* according to the network in the picture.
+ Restart the network service. If there are no errors, check that the machine address is correct with the `ip -4 a`command. Also ping ws22 from ws21. Similarly ping r1 from ws11.

#### 5.2. Enabling IP forwarding.
+ To enable IP forwarding, run the following command on the routers:
`sysctl -w net.ipv4.ip_forward=1`.

  *With this approach, the forwarding will not work after the system is rebooted.*

+ Open */etc/sysctl.conf* file and add the following line:
`net.ipv4.ip_forward = 1`

  *With this approach, IP forwarding is enabled permanently.*

#### 5.3. Default route configuration
Here is an example of the `ip r' command output after adding a gateway:
```
default via 10.10.0.1 dev eth0
10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.2
```

+ Configure the default route (gateway) for the workstations. To do this, add `default` before the router's IP in the configuration file
+ Call `ip r` and show that a route is added to the routing table
+ Ping r2 router from ws11 and show on r2 that the ping is reaching. To do this, use the `tcpdump -tn -i eth1`
command.

#### 5.4. Adding static routes
+ Add static routes to r1 and r2 in configuration file. Here is an example for r1 route to 10.20.0.0/26:
  ```shell
  # Add description to the end of the eth1 network interface:
  - to: 10.20.0.0
    via: 10.100.0.12
  ```

+ Call `ip r` and show route tables on both routers. Here is an example of the r1 table:
  ```
  10.100.0.0/16 dev eth1 proto kernel scope link src 10.100.0.11
  10.20.0.0/26 via 10.100.0.12 dev eth1
  10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.1
  ```
+ Run `ip r list 10.10.0.0/[netmask]` and `ip r list 0.0.0.0/0` commands on ws11.
+ Explain in the report why a different route other than 0.0.0.0/0 had been selected for 10.10.0.0/\[netmask\] although it could be the default route.

#### 5.5. Making a router list
Here is an example of the **traceroute** utility output after adding a gateway:
```
1 10.10.0.1 0 ms 1 ms 0 ms
2 10.100.0.12 1 ms 0 ms 1 ms
3 10.20.0.10 12 ms 1 ms 3 ms
```
+ Run the `tcpdump -tnv -i eth0` dump command on r1
+ Use **traceroute** utility to list routers in the path from ws11 to ws21
+ Based on the output of the dump on r1, explain in the report how path construction works using **traceroute**.

#### 5.6. Using **ICMP** protocol in routing
+ Run on r1 network traffic capture going through eth0 with the
`tcpdump -n -i eth0 icmp` command.

+ Ping a non-existent IP (e.g. *10.30.0.111*) from ws11 with the
`ping -c 1 10.30.0.111` command.

### Part 6. Dynamic IP configuration using **DHCP**

In this task you need to use ws1 and ws2 from [Part 5](#part-5-static-network-routing).

+ For r2, configure the **DHCP** service in the */etc/dhcp/dhcpd.conf* file:

  + specify the default router address, DNS-server and internal network address. Here is an example of a file for r2:
    ```shell
    subnet 10.100.0.0 netmask 255.255.0.0 {}

    subnet 10.20.0.0 netmask 255.255.255.192
    {
        range 10.20.0.2 10.20.0.50;
        option routers 10.20.0.1;
        option domain-name-servers 10.20.0.1;
    }
    ```
  + write `nameserver 8.8.8.8.` in a *resolv.conf* file
  + Restart the **DHCP** service with `systemctl restart isc-dhcp-server`. Reboot the ws21 machine with `reboot` and show with `ip a` that it has got an address. Also ping ws22 from ws21.

+ Specify MAC address at ws11 by adding to *etc/netplan/00-installer-config.yaml*:
`macaddress: 10:10:10:10:10:BA`, `dhcp4: true`
+ Сonfigure r1 the same way as r2, but make the assignment of addresses strictly linked to the MAC-address (ws11). Run the same tests
+ Request ip address update from ws21
+ Describe in the report what **DHCP** server options were used in this point.

### Part 7. **NAT**

In this task you need to use ws1 and ws2 from [Part 5](#part-5-static-network-routing).

+ In */etc/apache2/ports.conf* file change the line `Listen 80` to `Listen 0.0.0.0:80`on ws22 and r1, i.e. make the Apache2 server public
+ Start the Apache web server with `service apache2 start` command on ws22 and r1

+ Add the following rules to the firewall, created similarly to the firewall from Part 4, on r2:
  + delete rules in the filter table - `iptables -F`
  + delete rules in the "NAT" table - `iptables -F -t nat`
  + drop all routed packets - `iptables --policy FORWARD DROP`
+ Run the file as in Part 4
+ Check the connection between ws22 and r1 with the `ping` command
*When running the file with these rules, ws22 should not ping from r1*
+ Add another rule to the file:
  + allow routing of all **ICMP** protocol packets
+ Run the file as in Part 4
+ Check connection between ws22 and r1 with the `ping` command
*When running the file with these rules, ws22 should ping from r1*
+ Add two more rules to the file:
  + enable **SNAT**, which is masquerade all local ip from the local network behind r2 (as defined in Part 5 - network 10.20.0.0)
  *Tip: it is worth thinking about routing internal packets as well as external packets with an established connection*
  + enable **DNAT** on port 8080 of r2 machine and add external network access to the Apache web server running on ws22
  *Tip: be aware that when you will try to connect, there will be a new tcp connection for ws22 and port 80
+ Run the file as in Part 4
*Before testing it is recommended to disable the **NAT** network interface in VirtualBox (its presence can be checked with `ip a` command), if it is enabled*
+ Check the TCP connection for **SNAT** by connecting from ws22 to the Apache server on r1 with the `telnet [address] [port]` command
+ Check the TCP connection for **DNAT** by connecting from r1 to the Apache server on ws22 with the `telnet` command (address r2 and port 8080)

### Part 8. Bonus. Introduction to **SSH Tunnels**

In this task you need to use ws1 and ws2 from [Part 5](#part-5-static-network-routing).

+ Run a firewall on r2 with the rules from Part 7
+ Start the **Apapche** web server on ws22 on localhost only (i.e. in */etc/apache2/ports.conf* file change the line `Listen 80` to `Listen localhost:80`)
+ Use *Local TCP forwarding* from ws21 to ws22 to access the web server on ws22 from ws21
+ Use *Remote TCP forwarding* from ws11 to ws22 to access the web server on ws22 from ws11
+ To check if the connection worked in both of the previous steps, go to a second terminal (e.g. with the Alt + F2) and run the `telnet 127.0.0.1 [local port]` command.
