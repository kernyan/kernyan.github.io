---
layout: post
title:  "Leaking DNS Queries through VPN"
date:   2023-06-25 08:00:00 -0000
categories: FPGA, OS
---
When you have an active VPN connection, there has to be an intranet DNS server to resolve internal network url. Does the intranet DNS server get a chance to log public DNS queries?

# Table of contents
1. [Introduction](#introduction)
2. [Technical Background](#technical)
3. [Investigation](#investigation)
4. [Findings](#findings)

# Introduction <a name="introduction"></a>

When you connect to a VPN network, your machine and other machines on the VPN appears on the same local area network.

Common VPNs include commercial VPNs such as NordVPN, free ones like Hamachi, or simply your employers' VPN. They often show up as tun0 network interface on `ifconfig`

Practically, a subnet of your local ip range gets mapped to the internal network. E.g., 192.168.2.1 - 192.168.2.255, so that machines in the same LAN can directly address each other through their ip address. 

Remembering direct IP address is inconvenient so there's almost always an intranet DNS server that maps a local url to an ip. E.g., payroll.intranet_domain.com to 192.168.2.25 which is the intranet server hosting that webpage.

When you think about it, when there's a subnet remapping, there has to be two or more DNS servers doing url resolution as the public DNS server has no knowledge of the mapping on the intranet.

Here comes the privacy concern. If the order of DNS query resolution that the OS decides is always the intranet DNS server first, then your VPN provider would be able to log all url visits on your machine.

The rest of this article investigates whether this is true. 

<b>TLDR: On linux with systemd, DNS server consulted is listed on `/run/systemd/resolve/resolv.conf`. But because your OS does not wait for a response before trying other DNS servers, it's likely that all DNS servers receive your url.</b>

# Technical background <a name="technical"></a>
## DNS packets

When you type a url such as `www.google.com`, your OS encodes it in a DNS Packet, according to [rfc1035](https://datatracker.ietf.org/doc/html/rfc1035)

```rust
struct DnsPacket {
    header: DnsHeader,
    questions: Vec<DnsQuestion>,
    answers: Vec<DnsRecord>,
    authorities: Vec<DnsRecord>,
    additionals: Vec<DnsRecord>,
}
```

where `DnsQuestion`
```rust
struct DnsQuestion {
    qname: Vec<u8>,
    qtype: u16,
    qclass: u16,
}
```

The gist is that your OS crafts a bytes packet that encodes the url in a `DnsQuestion` structure, and sends it to a DNS Server. If the DNS server knows the IP mapping, it will return it, otherwise it will return the IP address of other DNS servers that might know.

## Public DNS

As your local machine does not know all the url-to-ip mappings in the world, it needs to query a public DNS server. The public DNS server you are using is typically provided by your ISP. You might have noticed that your smart TV, and router having a DNS server configuration, this is for when you prefer one that is different from the ISP provided. (Side note, some ad-blockers work by using a custom DNS server that simply does not resolve urls of advertisements)

A common public DNS server is one provided by OpenDNS, or Google at 8.8.8.8


# Investigation <a name="investigation"></a>

Now when you have an active VPN connection, some subnet address range is being remapped. The questions that we want to figure out are,
1. where is the DNS server configuration for both the public and internal DNS server located?, and
2. how does the OS decide which DNS server to send its DNS Query packets to, and in what order?

```bash
$ cat /etc/resolv.conf
nameserver 127.0.0.53
```

However this is not the one used. After some googling, when a VPN connection is active, systemd-resolve would regenerate a configuration with the additional intranet DNS server.

```bash
$ cat /run/systemd/resolve/resolv.conf
nameserver 192.168.2.1    # outgoing to ISP
nameserver 192.168.22.3   # intranet DNS
```

Now we know that systemd-resolve updates a resolv.conf on new VPN connection to include any additional DNS server it should consult.

My hypothesis is that it should be attempted in listed order, but let's just test it to be sure.

First we get the process listening at port 53, which is the default port for DNS server.

```bash
$ sudo lsof -n -i :53
COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 708 systemd-resolve   13u  IPv4  20617      0t0  UDP 127.0.0.53:domain 
systemd-r 708 systemd-resolve   14u  IPv4  20618      0t0  TCP 127.0.0.53:domain (LISTEN)
```

Next we shall log all system calls conducted by the process

```bash
$ sudo strace -o dns_query_order.log -p 708
```

At the same time, we trigger a DNS resolution on a url that we have never visited (to avoid DNS cache).

```bash
$ ping www.wonderfulcartoons.com
```

Here's the output from the log
```bash
$ grep 192.168 dns_query_order.log

connect(11, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("192.168.2.1")}, 16) = 0                               
connect(21, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("192.168.22.3")}, 16) = 0                              
connect(22, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("192.168.2.1")}, 16) = 0                               
connect(23, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("192.168.22.3")}, 16) = 0
recvmsg(23, {msg_name={sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("192.168.22.3")}, msg_namelen=128 => 16, msg_iov=[{iov_base="\363L\205\200\0\1\0\0\0\0\0\1\3www\26wonderful"...,  
recvmsg(11, {msg_name={sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("192.168.2.1")}, msg_namelen=128 => 16, msg_iov=[{iov_base="1\22\201\203\0\1\0\0\0\1\0\1\3www\26wonderful"..., 
```

You can see that
1. `systemd-resolve` did indeed query the DNS server listed first (192.168.2.1), however
2. it also queries the intranet DNS before it receives any response from the public DNS

which makes sense because there's no reason the OS should incur additional latency by doing the queries sequentially. And indeed, in this case, it receives a response from the intranet's DNS sooner.

# Findings <a name="findings"></a>

We conclude that
1. on a linux system with systemd, systemd-resolve regenerates `/run/systemd/resolve/resolv.conf` with the VPN's DNS server ip on a new VPN connection, and
2. on a url resolution, both DNS servers are queried because the OS does not wait for the first DNS server response's before trying the second one.

Therefore default systemd-resolve does not shield you from leaking url queries to VPN administrators. Conceivably, NordVPN or your employer could log url queries as long as the VPN connection is active.

You could write a custom DNS that queries the intranet with regex matches for intranet domains, but that's quite a bit of work.
