# File Analyze Tools
###### tags: `Security` `Tools`
[TOC]

---

# Reference
[GitHub - zardus](https://github.com/zardus/ctf-tools)

# Tools
## General
### xxd
```
0000000: 2e54 4820 5858 4420 3120 224d  .TH XXD 1 "M
000000c: 616e 7561 6c20 7061 6765 2066  anual page f
0000018: 6f72 2078 7864 220a 2e5c 220a  or xxd"..\".
0000024: 2e5c 2220 3231 7374 204d 6179  .\" 21st May
0000030: 2031 3939 360a 2e5c 2220 4d61   1996..\" Ma
000003c: 6e20 7061 6765 2061 7574 686f  n page autho
0000048: 723a 0a2e 5c22 2020 2020 546f  r:..\"    To
0000054: 6e79 204e 7567 656e 7420 3c74  ny Nugent <t
0000060: 6f6e 7940 7363 746e 7567 656e  ony@sctnugen
000006c: 2e70 7070 2e67 752e 6564 752e  .ppp.gu.edu.
```

### binwalk
從binary分析檔案(png + rar等等)

## Compressed File
### pkcrack
已知明文zip攻擊

## image
### stegsolve
圖片各種狀態
```shell=
wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve.jar
chmod +x stegsolve.jar
apt-get install openjdk-8-jre
java -jar stegsolve.jar 

java -version 
//openjdk version "1.8.0_131"
//OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-0ubuntu1.16.04.2-b11)
//OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```

### exiftool
```
apt-get install libimage-exiftool-perl
exiftool example.png
```

## Microsoft
### oletools
[GitHub](https://github.com/decalage2/oletools)
```
python3 -m pip install oletools
```
#### Tools to analyze malicious documents
- [oleid](https://github.com/decalage2/oletools/wiki/oleid): to analyze OLE files to detect specific characteristics usually found in malicious files.
- [olevba](https://github.com/decalage2/oletools/wiki/olevba): to extract and analyze VBA Macro source code from MS Office documents (OLE and OpenXML).
- [MacroRaptor](https://github.com/decalage2/oletools/wiki/mraptor): to detect malicious VBA Macros
- [msodde](https://github.com/decalage2/oletools/wiki/msodde): to detect and extract DDE/DDEAUTO links from MS Office documents, RTF and CSV
- [pyxswf](https://github.com/decalage2/oletools/wiki/pyxswf): to detect, extract and analyze Flash objects (SWF) that may
  be embedded in files such as MS Office documents (e.g. Word, Excel) and RTF,
  which is especially useful for malware analysis.
- [oleobj](https://github.com/decalage2/oletools/wiki/oleobj): to extract embedded objects from OLE files.
- [rtfobj](https://github.com/decalage2/oletools/wiki/rtfobj): to extract embedded objects from RTF files.

#### Tools to analyze the structure of OLE files
- [olebrowse](https://github.com/decalage2/oletools/wiki/olebrowse): A simple GUI to browse OLE files (e.g. MS Word, Excel, Powerpoint documents), to
  view and extract individual data streams.
- [olemeta](https://github.com/decalage2/oletools/wiki/olemeta): to extract all standard properties (metadata) from OLE files.
- [oletimes](https://github.com/decalage2/oletools/wiki/oletimes): to extract creation and modification timestamps of all streams and storages.
- [oledir](https://github.com/decalage2/oletools/wiki/oledir): to display all the directory entries of an OLE file, including free and orphaned entries.
- [olemap](https://github.com/decalage2/oletools/wiki/olemap): to display a map of all the sectors in an OLE file.

## virtual machine
### vhd (Virtual Hard Disk)
- 7z
    ```
    7z l ./9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
    ```
- guestmount
    ```
    apt install libguestfs-tools
    guestmount --add 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro -v /mnt/vhd
    ```

## zip file
### pkcrack
### fcrackzip
```
fcrackzip -D -p /usr/share/wordlists/rockyou.txt backup.zip
```