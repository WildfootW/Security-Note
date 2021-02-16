# Google Hacking
[TOC]
###### tags: `Security` `Tools`

## Alternate query types 備份查詢類型
### cache
`cache:www.csie.fju.edu.tw`
查詢某網頁在google中的cache

### link
`link:www.csie.fju.edu.tw`
查詢所有和網頁有連結的網頁

### related
`related:www.csie.fju.edu.tw`
列出所有和查詢網頁類似的網頁

### info
`info:hoschoc.com`
列出某網站在Google上存有哪些資訊
![](https://i.imgur.com/qlFMcUi.png)

## Other
### define
`define:赫蘿`
尋找某字詞在網路上的定義

## Query modifiers 查詢修飾符
### site
`Usage:"關鍵字 site:網址"`
查詢指定網站內的網頁
`site:tw`
也可以指定國籍

### intext
`Intext:台灣`
只想在網頁內文中尋找資料

### intitle
只想在網頁標題中尋找資料

### inurl
尋找指定的字串在網址列當中

### Filetype
`filetype:pdf`
`filetype:mp3`




## Advanced operators
|Operator|Purpose|Mixes with Other Operators?|Can be used Alone?|Web|Images|Groups|News|
|--- |--- |:---:|:---:|:---:|:---:|:---:|:---:|
|intitle|Search page Title|yes|yes|yes|yes|yes|yes|
|allintitle|Search page title|no|yes|yes|yes|yes|yes|
|inurl|Search URL|yes|yes|yes|yes|not really|like intitle|
|allinurl|Search URL|no|yes|yes|yes|yes|like intitle|
|filetype|specific files|yes|no|yes|yes|no|not really|
|intext|Search text of page only|yes|yes|yes|yes|yes|yes|
|allintext|Search text of page only|not really|yes|yes|yes|yes|yes|
|site|Search specific site|yes|yes|yes|yes|no|not really|
|link|Search for links to pages|no|yes|yes|no|no|not really|
|inanchor|Search link anchor text|yes|yes|yes|yes|not really|yes|
|numrange|Locate number|yes|yes|yes|no|no|not really|
|daterange|Search in date range|yes|no|yes|not really|not really|not really|
|author|Group author search|yes|yes|no|no|yes|not really|
|group|Group name search|not really|yes|no|no|yes|not really|
|insubject|Group subject search|yes|yes|like intitle|like intitle|yes|like intitle|
|msgid|Group msgid search|no|yes|not really|not really|yes|not really|