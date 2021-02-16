# SMB/SAMBA Enumeration & Attacks
[TOC]
###### tags: `Security` `Recon` `SMB` `SAMBA`

# Reference
* [Mad Irish - Hacking Windows shares from Linux with Samba](http://www.madirish.net/59)
* [SANS - Plundering Windows Account Info via **Authenticated** SMB Sessions](https://pen-testing.sans.org/blog/2013/07/24/plundering-windows-account-info-via-authenticated-smb-sessions)

# General
* `smbclient -U '' -L //10.10.10.184`
* `smbclient -U 'guest' -L //10.10.10.184`
* `smbclient -U 'anonymous' -L //10.10.10.184`

# Tools
## nbtscan
 ```
nbtscan -r 10.11.1.0/24
 ```
![](https://i.imgur.com/1gvGLPg.png)
 
## nmap NSE scripts
 ```
ls -l /usr/share/nmap/scripts/smb*
nmap -p 139, 445 --script=smb-os-discovery {{target ip}}
nmap -p 139, 445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 {{target ip}}
 ```
 
## enum4linux
It is written in Perl and is basically a wrapper around the Samba tools **smbclient**, **rpcclient**, **net** and **nmblookup**
 ```
enum4linux -a 10.11.1.227
 ```
![](https://i.imgur.com/LyOwe2f.png)
 
## nmblookup
```
nmblookup -A {{target ip}}
```
[Appendix C: Known NetBIOS **Suffix** Values](http://ubiqx.org/cifs/Appendix-C.html)
```
[name]      [suffix] - [group] [node type] [ACTIVE] [PERMANENT]
HPB4B52F0559C2  <00> -         B <ACTIVE> <PERMANENT> 
MSHOME          <00> - <GROUP> B <ACTIVE> <PERMANENT> 
HPB4B52F0559C2  <20> -         B <ACTIVE> <PERMANENT> 
HP0559C2        <00> -         B <ACTIVE> <PERMANENT> 
HP0559C2        <20> -         B <ACTIVE> <PERMANENT> 
```
![](https://i.imgur.com/2fnKk0i.png)
## smbclient (null session)
```
smbclient -L {{netbios name}} -I {{target ip}} -U {{username}}
smbclient //{{server}}/{{service}} -I {{target ip}} -U {{username}}
```
## rpcclient (111/135)
## Metasploit SMB auxiliary scanners
[SCANNER SMB AUXILIARY MODULES](https://www.offensive-security.com/metasploit-unleashed/scanner-smb-auxiliary-modules/)

## mount
```
mkdir /mnt/smb
mount -t cifs -o username=guest //10.10.10.134/Backups /mnt/smb
```