# Web Information Gathering
[TOC]
###### tags: `Security` `Web` `Recon`

# Certificate
- Domain Enumeration (憑證綁定域名)
- [crt.sh](https://crt.sh)

# information leak
- Username / Password
- Source Code
- Url Route / File Path
- Config

## What targets?
Backup / Version Control / Temporary files
- .git [scrabble](https://github.com/denny0223/scrabble), [GitHack](https://github.com/lijiejie/GitHack)
- .svn
- xxx.php.bak / www.tar.gz / .xxx.php.swp / xxx.php~ / xxx.phps
- .pyc
- robots.txt
- /.xxx
- .DS_Store [DS_Store_exp](https://github.com/lijiejie/ds_store_exp)
- .htaccess
- package.json
- server-status
- crossdomain.xml
- admin/ manager/ login/ backup/ wp-login/ phpMyAdmin/
- /WEB-INF/web.xml

### config
- apache2 configuration
    identify the path of the webroot.
    `/etc/apache2/sites-enabled/000-default.conf`
 
## methods

### use specific domain
edit `/etc/hosts`

### Web path scanner
#### gobuster
```
gobuster dir               \
-w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt \
-x php,html                \
-u http://x.x.x.x/         \
-o dirbust-root.log
```
* `-x string` File extension(s) to search
#### [dirsearch](https://github.com/maurosoria/dirsearch)

#### [DirBuster](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project)


# Fingerprint Scanning
* wafw00f - web application firewall detector

# Vulnerability Scanning
#### Nikto
Web server scanner
```
perl nikto.pl -h {{192.168.0.1}} -p {{80,88,443}}
perl nikto.pl -update
```
#### WPScan
Wordpress scanner

# firefox addons
* ### flagfox
displays a flag depicting the location of the current server
* ### wappalyzer
identifies software on the web
