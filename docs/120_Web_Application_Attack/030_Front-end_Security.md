# Web Front-end Security
[TOC]
###### tags: `Security` `Web`

# Same Origin Policy 同源政策 (SOP)
瀏覽器的安全策略，同協議 && 同域名 && 同端⼝ 才算是伺服器能讀取得自己的資源
有些例外的 `tag` : `<script>` `<img>` `<link>` `<iframe>` ...

## 通過的管道
### JSONP (JSON with Padding)
利用 `<script>` 可以跨域的特性來傳 JSON 資料

### CORS (Cross-Origin Resource Sharing)
[mozilla](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS)
用 Header 的內容跟瀏覽器表示信任某個 domain，讓他進行跨域存取
![](https://i.imgur.com/4JCKeC8.png)
