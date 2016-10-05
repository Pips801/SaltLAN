# SaltLAN

Single-server LAN party

## What

The goal of SaltLAN's network and server infrastructure was designed to be small, single-system, and cheap, while meeting all of our requirements: Caching, Game servers, monitoring, and 100Mb/s to the client.

I wanted to build SaltLAN after attending a LAN party in May, 2016. While the LAN was a lot of fun and it featured a great giveaway, awesome unlimited drinks and cool people, however, there were some massive networking problems.

 1. No dedicated LAN servers for games like TF2. People had to try and host local games, but nobody could see/connect to the game.
 2. Bad switches/using home routers instead of an unmanaged switch with distribution switches. Nobody could talk to each other.
 3. Slow internet (20Mbps max). Not everybody had certain games, and the day of the LAN, there was a massive TF2 update. It killed the network several times.
 
 
## Configuration

###Server

Component | Hardware
--- | ---
**CPU** |Xeon E3 1220v5 
**RAM** |32GB DDR4      
**Storage** | 512GB Samsung EVO SSD
  | 3TB WD BLACK HDD
**Networking**| 3x 1Gbps NICS
  | 1x 1Gbps NIC for WAN
  | 2x 1Gbps NICs bonded for 2Gbps into LAN
  
###Network

Component | Hardware
--- | ---
**Core switch** | 1x Cisco SLM 2048 48 port Gigabit switch
**Disribution switches** | 6+x Netgear Prosafe 24 port 10/100 switch with 2x 1Gbps uplink

###Software


OS: Ubuntu server 16.04

DHCP: dnsmasq

DNS: dnsmasq

Caching: Docker







