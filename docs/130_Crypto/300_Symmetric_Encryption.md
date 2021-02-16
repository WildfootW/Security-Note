# Symmetric Encryption
[TOC]
###### tags: `Security` `Crypto`

### Xor cipher
    xortool : https://github.com/hellman/xortool
    most frequent byte:
    * binary 00
    * text 20(space)https://zh.wikipedia.org/wiki/ASCII
### DES
using ssl
```
openssl enc –h
列出 OpenSSL 提供的對稱式加解密演算法

openssl des -in file -out file.des
使用 DES 加密

openssl des -d -in file.des -out file
使用 DES 解密
```
### Triple DES
### AES
* Padding oracle attack
    伺服器能幫我們解密任意訊息 但我們只能知道有沒有 padding 錯誤
