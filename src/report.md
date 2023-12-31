## Part 1. ipcalc tool
1. network address of `192.167.38.54/13` is `192.160.0.0`
1. conversion of:
   
    + the mask `255.255.255.0` to prefix and binary is `/24` and `11111111.11111111.11111111.00000000`
    + `/15` to normal and binary is `255.254.0.0` and `11111111.11111110.00000000.00000000`
    + `11111111.11111111.11111111.11110000` to normal and prefix is `255.255.255.240` and `/28`

1. minimum and maximum host in `12.167.38.4` network with masks:
    + `/8` is `12.0.0.1` and `12.255.255.254`
    + `11111111.11111111.00000000.00000000` is `12.167.0.1` and `12.167.255.254`
    + 255.255.254.0 is `12.167.38.1` and `12.167.39.254`
    + `/4` is `15.255.255.254` and `15.255.255.255`

1. an application running on localhost
   + can be accessed with the following IPs: `127.0.0.2`, `127.1.0.1`
   + can't be accessed with the following IPs: `194.34.23.100`, `128.0.0.1`
1. IPs that can be used as public:
    + `134.43.0.2`, `172.0.2.1`, `192.172.0.1`, `172.68.0.2`, `192.169.168.1`
  
    IPs that can only be used as private:

    + `10.0.0.45`, `192.168.4.2`, `172.20.250.4`, `172.16.255.255`, `10.10.10.10`

2. possible gateway IP addresses for 10.10.0.0/18 are: `10.10.0.2`, `10.10.10.10`, `10.10.1.255` (`10.0.0.1`, `10.10.100.1` are not possible). 

## Part 2. Static routing between two machines

1. output of `ip a` command on `ws1` and `ws2`

    ![ip a on ws1](img/p2_1.png)

    ![ip a on ws2](img/p2_2.png)

1. `enp0s8` is the local network between `ws1` and `ws2`

    ![config of ws1](img/p2_4.png)

    ![config of ws2](img/p2_3.png)

    restart of the network services:

    ![restart ws1](img/p2_6.png)

    ![restart ws2](img/p2_5.png)


2. static routes with `ip r` add
  
    ![ip r add ws1](img/p2_8.png)

    ![ip r add ws2](img/p2_7.png)

1. static routes with `netplan config`

    ![netplan ws1](img/p2_10.png)

    ![netplan ws2](img/p2_9.png)
    
## Part 3. iperf3 utility

1. 8 Mbps = 1 MB/s
   
   100 MB/s = 800000 Kbps

   1 Gbps = 1000 Mbps

1. `iperf` server running on `ws2`
   
   ![iperf on ws2](img/p3_2.png)

   `iperf` client running on `ws1`

   ![iperf on ws1](img/p3_1.png)

## Part 4. Network firewall

1. firewall on `ws1`
    
    ![firewall ws1](img/p4_2.png)
    
    ![firewall ws2](img/p4_11.png)

    
    firewall on `ws2`
    
    ![firewall ws2](img/p4_1.png)
    
    ![firewall ws1](img/p4_22.png)

2. The difference is that the `first` found rule applies, so for `ws1` pings will be be dropped and for `ws2` will not.

3. output of `ping` and `nmap` for `ws1` and `ws2`
   
   ![ping nmap on ws1](img/p4_4.png)

   ![ping nmap on ws2](img/p4_3.png)

## Part 5. Static network routing

1. `ws22` machine configuration
   
   ![ws22](img/p5_1_ws22.png)

   `r1` machine configuration
   
   ![r1](img/p5_1_r1.png)

   `r2` machine configuration
   
   ![r2](img/p5_1_r2.png)

    `ws11` machine configuration and `ping` to `r1`
   
   ![ws11](img/p5_1_ws11.png)

   `ws21` machine configuration and `ping` to `ws22`
   
   ![ws21](img/p5_1_ws21.png)

1. enabling IP forwarding via `sysctl` 
   
   ![r1](img/p5_2_1_r1.png)

   ![r2](img/p5_2_1_r2.png)

   enabling IP forwarding via `/etc/sysctl.conf`

   ![r1](img/p5_2_2_r1.png)

   ![r2](img/p5_2_2_r2.png)

1. `ws11` default route configuration:
   
    ![ws11](img/p5_3_ws11.png)

    `ws21` default route configuration:
   
    ![ws21](img/p5_3_ws21.png)

    `ws22` default route configuration:
   
    ![ws22](img/p5_3_ws22.png)

    ping `r2` router from `ws11`

    ![ws11](img/p5_3_ws11_ping.png)

    ![r2](img/p5_3_r2.png)

1. static route to `10.20.0.0` on `r1`
   
   ![r1](img/p5_4_r1.png)

   static route to `10.10.0.0` on `r2`
   
   ![r2](img/p5_4_r2.png)

   `ip r list 10.10.0.0/18` and `ip r list 0.0.0.0/0` commands on `ws11`

   ![ip r list](img/p5_4_ws11.png)

   the route 10.10.0.0/18 is selected because it's more specific (longer prefix), also packets will be routed via the default route, if the routing table has no other routes, that can be used, which is not the case here.

1. `traceroute` call from `ws11` to `ws21`
   
   ![ws11](img/p5_5_ws11.png)

   `tcpdump` on `r1`

   ![r1](img/p5_5_r1.png)

   `Traceroute` is used to track the `path` of a packet on an IP network from source to destination.
   
   According to dump we can see that it uses `"Time to Live"` (`TTL`) mechanism. The source sending packets with increasing `TTL` for each hop until it gets to the destination (from `1` to `3` in the dump). The `TTL` value specifies how many routers the packet can pass through before being discarded. With each passed route `TTL` decreases.
   
   After the packet is discarded (`TTL` after decreasing equals `0`) the destination sends a packet to the source with `"ICMP time exceeded"` if the packet didn't reach the destination and `"Echo Reply"` if the destination is reached. 
   
   Traceroute also uses `3` packets on each hop to get a more accurate average of the round-trip time. You can change the amount of packets with `-q` option.

1. traffic capture going through `eth0` on `r1`
   
   ![r1](img/p5_6_r1.png)

   ping a non-existent IP `10.30.0.111` from `ws11`:

   ![ws11](img/p5_6_ws11.png)

## Part 6. Dynamic IP configuration using DHCP

1. configure the DHCP service for `r2` and change nameserver to 8.8.8.8 in resolv.conf
   
    ![r2 dhcp](img/p6_1_r2.png)

    restart `DHCP`

    ![r2 dhcp restart](img/p6_2_r2.png)

1.  `ws21` address after reboot and ping to ws22:

    ![ws21 ip a](img/p6_3_ws21.png)

1. Specify MAC address at ws11
   
   ![ws11 mac](img/p6_4_ws11.png)

1. configure `r1`
   
   ![r1 dhcp](img/p6_5_r1.png)

    and restart DHCP 

   ![r1 dhcp restart](img/p6_6_r1.png)

1. `ws11` address after reboot and ping to `ws22`:
   
   ![ws11 ip a](img/p6_7_ws11.png)

1. request ip address update from ws21

    ![ws21 update](img/p6_8_ws21.png)

1. in the part 6 we used these dhcp server options:
   
   + `subnet`: definition of subnets
   + `range`: tells DHCP that it can dynamically assign any IP address within the specified range (inclusive)
   + `option routers`: sets the default gateway for the subnet (similar to the rule `- to: default via: ...`)
   + `option domain-name-servers`: sets the DNS server for the subnet
   + `host`: specify a certain IP address tied to a mac address (also known as static lease or reservation)


## Part 7. NAT

1. changing apache config on `r1` and `ws22`:
   
    ![r1 apache](img/p7_1_r1.png)

    ![ws22 apache](img/p7_1_ws22.png)

    and starting the server:

    ![r1 start apache](img/p7_2_r1.png)

    ![ws22 start apache](img/p7_2_ws22.png)

1. firewall rules on `r2`:

    ![r2 firewall](img/p7_3_r2.png)

    then check connection between `ws22` and `r1`:
   
   ![ping ws22 from r1](img/p7_4_r1.png)

1. add a rule allowing of all ICMP protocol packets on `r2`
   
   ![r2 firewall](img/p7_5_r2.png)

    then check connection again

   ![ping ws22 from r1](img/p7_5_r1.png)

1. SNAT and DNAT rules on `r2`
   
   ![snat dnat r2](img/p7_6_r2.png)

    telnet from `ws22` to the apache server on `r1`

   ![telnet r2](img/p7_7_ws22.png)

    telnet from `r1` to the apache server on `ws22`

   ![telnet r1](img/p7_7_r1.png)

## Part 8. Bonus. Introduction to SSH Tunnels

1. edit `/etc/apache2/ports.conf` on `ws22` and run apache server
   
    ![apache ws22](img/p8_1_ws22.png)

1. Local TCP forwarding from `ws21` to `ws22` to access the web server on `ws22` from `ws21`
   
    ![local ssh](img/p8_2_ws21.png)

    ![local ssh](img/p8_3_ws21.png)

    ![local ssh](img/p8_4_ws21.png)

    In short a local ssh tunneling  is used to forward a port from a client to a server. A client connects to a server, and then, connections made to a specified port on the client side are forwarded, via the SSH connection, to a specified port on the server side

2. Remote TCP forwarding from `ws11` to `ws22` to access the web server on `ws22` from `ws11`
   
    ![remote ssh](img/p8_5_ws22.png)

    ![remote ssh](img/p8_6_ws22.png)

    ![remote ssh](img/p8_7_ws22.png)

    In short a remote ssh tunneling is used to forward a port from a server to a client (opposite to local ssh tunneling). A client connects to a server, and then, connections made to a specified port on the server side are forwarded, via the SSH connection, to a specified port on the client side.


