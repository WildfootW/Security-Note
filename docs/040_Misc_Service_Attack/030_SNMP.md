# SNMP Enumeration
[TOC]
###### tags: `Security` `Recon` `SNMP`

# Tools
## nmap
```
nmap -sU --open -p 161 10.11.1.1-254 -oG mega-snmp.txt
```
* find SNMP on UDP port
    * `nmap -sU -sC -oA nmap.udp.log 10.10.10.20`
    * `nmap -sU -sC -n -p 161 -oA udp 10.10.10.20`

## onesixtyone (Fast SNMP Scanner)
* `onesixtyone 10.10.10.0/24 public`
 
## snmpwalk
* Tool - `snmpwalk -v2c -c public 10.10.10.20`
    * `iso.3.6.1.2.1.4.34.1.3.1.4`
    * `iso.3.6.1.2.1.4.34.1.3.2.16`
* Getting SNMP MIBs Installed and Configured
    * `apt install snmp-mibs-downloader`
    * `vim /etc/snmp/snmp.conf`
    * ![](https://i.imgur.com/8mX6E5W.png)
```
# Enumerating the Entire MIB Tree
snmpwalk -c public -v1 10.11.1.219
# Enumerating Windows Users
snmpwalk -c public -v1 10.11.1.204 1.3.6.1.4.1.77.1.2.25
# Enumerating Running Windows Processes
snmpwalk -c public -v1 10.11.1.204 1.3.6.1.2.1.25.4.2.1.2
# Enumerating Open TCP Ports
snmpwalk -c public -v1 10.11.1.204 1.3.6.1.2.1.6.13.1.3
# Enumerating Installed Software
snmpwalk -c public -v1 10.11.1.204 1.3.6.1.2.1.25.6.3.1.2
```
```
cat snmpwalk.log | grep IP-FORWARD-MIB::inetCidrRouteIfIndex.ipv6    
IP-FORWARD-MIB::inetCidrRouteIfIndex.ipv6."de:ad:be:ef:00:00:00:00:02:50:56:ff:fe:b9:c9:36".128.3.0.0.7.ipv6."00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00" = INTEGER: 1
```

## Enyx
* Tool - Enyx - SNMPv6 Enumeration via Python