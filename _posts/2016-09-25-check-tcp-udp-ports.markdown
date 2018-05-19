---
layout: post
title:  "Check TCP or UDP ports using netcat (nc) command"
date:   2016-09-25 10:51:47 +0530
categories: Linux
image: /images/netcat.png
categories: [one, two]
---


netcat or nc is a simple unix utility which reads and writes data across network connections, using TCP or UDP protocol. It is designed to be a reliable "back-end" tool that can be used directly or easily driven by other programs and scripts.  At the same time, it is a feature-rich network debugging and exploration tool, since it can create almost any kind of connection you would need and has several interesting built-in capabilities.

Syntax of netcat command is:

```
netcat [options] host port
or
nc [options] host port
```

Here we will be discussing only about the port scanning feature of netcat command. In order to check whether a TCP port is open, we usually make use of the telnet command. But telnet cannot be used to check UDP ports as telnet only works with tcp protocol. So in that case we can make use of the netcat command.

**Check TCP port:**

Following is the syntax to check tcp ports:

```
nc -zv <domain_name or ip-address> <port>
```


where
 -z      Specifies that nc should just scan for listening daemons, without sending any data to them.
 -v      Have nc give more verbose output.

*For example:*
Suppose our test server has IP address 192.168.0.10.

Lets take a look at a TCP port 80 in this server. Here apache service is listening on port 80:

```
root@server:~# hostname -i
192.168.0.10

root@server:~# netstat -ntpl | grep :80
tcp       0      0 0.0.0.0:80                   0.0.0.0:*                    LISTEN      5272/apache2   
```

Now from our client machine we can check port 80:

```
root@client:~# nc -zv 192.168.0.10 80
Connection to 192.168.0.10 80 port [tcp/http] succeeded!
```

Here we see from the output that the server 192.168.0.10 has apache running on port 80.

**Check UDP port:**

Following is the syntax to check UDP ports:

```
nc -zuv <domain_name or ip-address> <port>
```

where
 -u      Use UDP instead of the default option of TCP.

Lets take a look at a udp port 123 in this server. Here the ntpd service is listening on port 123:

```
root@server:~# hostname -i
192.168.0.10
root@server:~# netstat -ntupl | grep :123     
udp        0      0 127.0.0.1:123           0.0.0.0:*                           6582/ntpd       
udp        0      0 0.0.0.0:123             0.0.0.0:*                           6582/ntpd        
```

Now from our client machine we can check the port 123:

```
root@client:~# nc -zuv 192.168.0.10 123
Connection to 192.168.0.10 123 port [udp/ntp] succeeded!
```

Here we see from the output that the server 192.168.0.10 has ntp running on port 123 which make use of the udp protocol.


**Scan muliple ports:**

We can use netcat to scan for multiple ports by specifing each port in the nc command:

```
nc -zv <ip-address> <port-1> <port-2>...<port-n>
```

*Example:*

```
root@dan-desktop:~# nc -zv 192.168.0.10 21 22 80
Connection to 192.168.0.10 21 port [tcp/ftp] succeeded!
Connection to 192.168.0.10 22 port [tcp/ssh] succeeded!
Connection to 192.168.0.10 80 port [tcp/http] succeeded!
```

**Scan range of ports:**

Netcat can also be used to scan a range of ports. Like if you want to scan ports from 1 to 500 we can use the following command:

```
nc -zv <ip-address> <port-range>
```

*Example:*

```
root@client:~# nc -zv 192.168.0.10 1-500
nc: connect to 192.168.0.10 port 1 (tcp) failed: Connection refused
nc: connect to 192.168.0.10 port 2 (tcp) failed: Connection refused
nc: connect to 192.168.0.10 port 3 (tcp) failed: Connection refused
nc: connect to 192.168.0.10 port 4 (tcp) failed: Connection refused
nc: connect to 192.168.0.10 port 5 (tcp) failed: Connection refused
nc: connect to 192.168.0.10 port 6 (tcp) failed: Connection refused
.
.
.
Connection to 192.168.0.10 21 port [tcp/ftp] succeeded!
Connection to 192.168.0.10 22 port [tcp/ssh] succeeded!
nc: connect to 192.168.0.10 port 23 (tcp) failed: Connection refused
nc: connect to 192.168.0.10 port 24 (tcp) failed: Connection refused
.
.
```

From the above output we can see that the open ports are 21, 22 and 80.

We can also filter the ouput to show only the open ports using following:

```
root@dan-desktop:~# nc -zv 192.168.0.10 1-500 2>&1 | grep succeeded
Connection to 192.168.0.10 21 port [tcp/ftp] succeeded!
Connection to 192.168.0.10 22 port [tcp/ssh] succeeded!
Connection to 192.168.0.10 80 port [tcp/http] succeeded!
```
