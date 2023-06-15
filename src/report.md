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

