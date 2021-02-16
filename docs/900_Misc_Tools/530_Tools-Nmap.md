# Nmap
[TOC]
###### tags: `Security` `Tools`

# Cheet sheet
## Initial Scan
```SHELL
nmap -p- -vvv -oN nmap.scan_all_port.log {{target}}
nmap -A -sC -sV -oN nmap.initial.log {{target}}
cat nmap.scan_all_port.log | grep "^[0-9]" | awk -v FS="/" '{print $1}' | tr '\n' ',' | sed s/,$//
```
```SHELL
ports=$(nmap -p- --min-rate=1000 -T4 {{target}} | grep "^[0-9]" | cut -d '/' -f 1 |
tr '\n' ',' | sed s/,$//)
nmap -sC -sV -p $ports {{target}}
```
tip: `sleep 300;`
## Vulnerable Scan
```SHELL
nmap --script "vuln" -oN nmap.vulnscan.log {{target}}
nmap --script="smb-vuln*" -p 139, 445 -v -oN nmap.vulnscan.smb.log {{target}}
```

## Others
```SHELL
nmap -p- -sT {{target}}
nmap -sT -sV -A -O -p 1-65535 {{target}}
```
# Parameters
* `-A`
Enable OS detection (need sudo), version detection, script scanning, and traceroute

## Output
* `-oA {{filename}}`
Output in the three major formats at once
* `-oN {{filename}}`
Output scan in normal format

## speed
* `-T paranoid(0)|sneaky(1)|polite(2)|normal(3)|aggressive(4)|insane(5)`
Set a timing template

* `--min-rate {{number}}` `--max-rate {{number}}`
`--min-rate 300` means that Nmap will try to keep the sending rate at or above 300 packets per second.

```SHELL
-sn # Ping Scan - disable port scan
-sT(TCP) | -sU(UDP) | -sY(SCTP INIT) | -sA(TCP ACK)
-sN(Null) | -sF(FIN) | -sX(Xmas)

--top-ports={{number}}
# 掃描前 {{number}} 常用的 port

-sV 
# enumerate versions

-sC --script=default 
# run safe scripts

-Pn 
# skip host discovery

-p {{port}} {{ranges}}
# -p 139,445
# -p 1-65535 == -p - # 範圍的頭尾如果是 1/65535 可省略
# -p U:53,111,137,T:21-25,80 # T: for TCP, U: for UDP, S: for SCTP, or P: for IP Protocol

--open 
# show only open ports
```


* `-v | -vv | -vvv`
verbosity level

# Guide
## Dealing with Misidentified and Unidentified Hosts
如果沒辦法準確的偵測 OS，有幾個做法：
* 更新到最新版本
* 掃描更多 port
    z.B. `-p-` 掃描全部的 port 或 `-sU` 加上 UDP Scan
* 使用更積極的猜測
    有些時候會因為防火牆之類的問題得到不同的結果 加上 `--osscan-guess` 參數
* 換個地方進行偵測

## Updater for Nmap's architecture-independent files
因為 nselib 和 nse scripts 常常需要更新，所以我現在的做法就是直接 clone nmap 的 GitHub 下來，link nselib 和 scripts 到 `/usr/share/nmap` (把原本的 `mv` 到 `*.old`) 
另外在 [manual](http://man7.org/linux/man-pages/man1/nmap-update.1.html) 有看到一個叫做 `nmap-update` 的東西看起來是我要的，但目前 (7.80) 似乎還沒有這個功能