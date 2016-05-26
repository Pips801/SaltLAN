# SaltLAN

Single-server LAN party

## What

Alright, I've wanted to write this down for a while. SaltLAN is a self-contained LAN party, featuring Routing, DHCP, DNS, Game servers and game download caching.

I wanted to build SaltLAN after attending a LAN party in May, 2016. While the LAN was a lot of fun and it featured a great giveaway, awesome unlmited drinks and cool people, however, there were some massive networking problems.

 1. No dedicated LAN servers for games like TF2. People had to try and host local games, but noby could see/connect to the game.
 2. Bad switches/using home routers instead of an unmanaged switch with distrobution switches. Nobody could talk to eachother.
 3. Slow internet (20Mbps max). Not everybody had certan games, and the day of the LAN, there was a massive TF2 update. It killed the network several times.
 
 After talking to some people in the freenode/#DC801 chanel about building a portable LAN party, I was ready to start.
 
## pre-rec

Everything is currently being run on a (kinda crappy) Supermicro server. 2x Xeons, 10GB DDR2, 2x500GB HDDs in RAID0, and dual Gigabit NICs.

I have plans on moving it into a short-depth server with a single Xeon, 16-32GB DDR3, 1TB in RAID1 and dual 126GB SSDs in RAID0 for caching.

## Setup

FYI, I am not a Linux guy, so at some point I'll probably say/do something stupid that has a much easier solution. I don't care!

* Ubuntu Server 16.04 with GNOME desktop
* NIC 1 (`eth0`) -> `enp4s0f0` and NIC 2 (`eth1`) -> `enp4s0f1`

## Steps
1. Install linux. I'm not going to walk you through this, you can do it yourself.
2. Set up routing.
3. Set up DHCP.
5. Set up steamcache and lancache.
6. Set up DNS server.
7. Set up gameservers.
8 (optional) Load up cache with games.

## Install
Okay, I should shut up and tell you how.

Once linux is installed, we need to turn it into a router.

We have two NICs, try to remember which one gets plugged into the wall (WAN) and what gets plugged into the switch (LAN). For me, it was `eth0` is WAN and `eth1` is LAN.

### Interface configuration

`nano /etc/network/interfaces`

It should look like this:
```
auto lo
iface lo inet loopback
```

**Note:** I am going to be replacing enp4s0fX with eth0 and eth1. check ifconfig for what your interfaces are actually called)

We need to configure the NICs now. add these lines so the file looks like this. (**make sure to replace certan bits with whatever your setup is**)


```
# WAN - to wall
auto eth0
iface eth0 dhcp

# LAN - to switch
auto eth1
iface eth1 inet static
    address 10.0.0.1 # address the router will be on the LAN
    netmask 255.255.255.0 # netmask for the network. You can change this if you know what yo're doing.
    network 10.0.0.0 # base address for the network
    broadcast 10.0.0.255 # broadcast address
```

### IPv4 Forwarding
Alright, good. Now we need to do IPv4 forwarding.

`nano /etc/sysctl.conf`

Look for the line
`# net.ipv4.ip_forward=1`
and un-comment it so it looks like
`net.ipv4.ip_forward=1`

Save and exit.

We want to turn on Ip forwarding right now as well, so we don't have to reboot. 

run
`sysctl -w net.ipv4.ip_forward=1`

### iptables

Now we need to enable iptables

`nano /etc/rc.local`

and add these lines **BEFORE** the `exit 0` line.

```
/sbin/iptables -P FORWARD ACCEPT
/sbin/iptables --table nat -A POSTROUTING -o eth0 -j MASQUERADE
```

To make it work without rebooting, run

`iptables -P FORWARD ACCEPT` and `iptables –-table nat -A POSTROUTING -o eth0 -j MASQUERADE`

# DHCP server

Good, now we can install the DHCP server.
 
 `apt-get install isc-dhcp-server`
 
 Now we need to configure the DHCP server. Later on we will be using the steamcache-dns docker image as our DNS server.
 
 `nano /etc/dhcp/dhcpd.conf`
 
 then configure it to look like this
 
```
# set the dns server to our internal server (docker dns server)
option domain-name-servers 10.0.0.1 ;

default-lease-time 3600;
max-lease-time 7200;
ddns-update-style none;

# set up the DHCP configuration. Change this if you know networking.
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.10 10.0.0.254;
  option subnet-mask 255.255.255.0;
  option broadcast-address 10.0.0.255;
  option routers 10.0.0.1;
}
```

Now we want to set it to run the DHC server **only** on the LAN (not the WAN)

`nano /etc/default/isc-dhcp-server`

The line will look like this before you change it
`INTERFACES=""`
Add in the interface that's running the LAN
`INTERFACES="eth1"`

Now we need to restart the DHCP server.

`service isc-dhcp-server restart`

