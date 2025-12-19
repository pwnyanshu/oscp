
FTP is **a file-moving protocol from the 1970s**.

Its entire job:
> “Move files between two machines.”

FTP is **plaintext**. So Man/Women in the middle can see stuff.

---

FTP uses TWO connections, not one.

### 1. Control channel (always port 21)

This is where:

- USER
- PASS
- LIST
- RETR
- STOR

commands go.

Think of it as **chatting instructions**, not data.


### 2. Data Chanel

And this channel behaves _differently_ depending on **Active vs Passive mode**.

#### Active
Here Client send the FTP server request to connect on port 21, Server initiate a connection.
But it never reaches the client because the clients firewall blocks the incoming connection.

Thats why we have Passive connection

#### Passive
- Client connects to server on port 21
- Client says: `PASV`
- Server replies:
    > “Cool, connect to me on port Y”
- Client initiates **both** connections

Works because outgoing connection are allowed for/by the client.

---

## Default Configuration


One of the most used FTP servers on Linux-based distributions is [vsFTPd](https://security.appspot.com/vsftpd.html).
The default configuration of vsFTPd can be found in `/etc/vsftpd.conf`

vsftpd = **Very Secure FTP Daemon**


```shell-session
# Installation
sudo apt install vsftpd 

# vsFTPd Config File
cat /etc/vsftpd.conf | grep -v "#"
```


|**Setting**|**Description**|
|---|---|
|`listen=NO`|Run from inetd or as a standalone daemon?|
|`listen_ipv6=YES`|Listen on IPv6 ?|
|`anonymous_enable=NO`|Enable Anonymous access?|
|`local_enable=YES`|Allow local users to login?|
|`dirmessage_enable=YES`|Display active directory messages when users go into certain directories?|
|`use_localtime=YES`|Use local time?|
|`xferlog_enable=YES`|Activate logging of uploads/downloads?|
|`connect_from_port_20=YES`|Connect from port 20?|
|`secure_chroot_dir=/var/run/vsftpd/empty`|Name of an empty directory|
|`pam_service_name=vsftpd`|This string is the name of the PAM service vsftpd will use.|
|`rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`|The last three options specify the location of the RSA certificate to use for SSL encrypted connections.|
|`rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key`||
|`ssl_enable=NO`||

In addition, there is a file called `/etc/ftpusers` that we also need to pay attention to, as this file is used to deny certain users access to the FTP service.

```shell-session
cat /etc/ftpusers
```

---

## Dangerous Settings

### Anonymous FTP


Anonymous FTP means:

- Username: `anonymous`
- Password: anything (or nothing)

Admins enable this for “convenience”.

Attackers love it because:

- No creds required
- Often misconfigured
- Files accidentally exposed
- Sometimes **write access enabled** (catastrophic)

Anonymous + write access =

> “Please upload your web shell here.”

|**Setting**|**Description**|
|---|---|
|`anonymous_enable=YES`|Allowing anonymous login?|
|`anon_upload_enable=YES`|Allowing anonymous to upload files?|
|`anon_mkdir_write_enable=YES`|Allowing anonymous to create new directories?|
|`no_anon_password=YES`|Do not ask anonymous for password?|
|`anon_root=/home/username/ftp`|Directory for anonymous.|
|`write_enable=YES`|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|

```shell-session
ftp 10.129.14.136

# Directory listing
ftp> ls

# to get the first overview of the server's settings, we can use the following command:
ftp> status

# Some commands should be used occasionally, as these will make the server show us more information that we can use for our purposes. These commands include `debug` and `trace`.
ftp> debug
ftp> trace
ftp> ls
```


|**Setting**|**Description**|
|---|---|
|`dirmessage_enable=YES`|Show a message when they first enter a new directory?|
|`chown_uploads=YES`|Change ownership of anonymously uploaded files?|
|`chown_username=username`|User who is given ownership of anonymously uploaded files.|
|`local_enable=YES`|Enable local users to login?|
|`chroot_local_user=YES`|Place local users into their home directory?|
|`chroot_list_enable=YES`|Use a list of local users that will be placed in their home directory?|

|**Setting**|**Description**|
|---|---|
|`hide_ids=YES`|All user and group information in directory listings will be displayed as "ftp".|
|`ls_recurse_enable=YES`|Allows the use of recurse listings.|

---

In the following example, we can see that if the `hide_ids=YES` setting is present, the UID and GUID representation of the service will be overwritten, making it more difficult for us to identify with which rights these files are written and uploaded.

#### Hiding IDs - YES

  FTP

```shell-session
ftp> ls

---> TYPE A
200 Switching to ASCII mode.
ftp: setsockopt (ignored): Permission denied
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

**This setting is a security feature to prevent local usernames from being revealed. With the usernames, we could attack the services like FTP and SSH and many others with a brute-force attack in theory.**

---
#### Download a File

```shell-session
ftp> get Important\ Notes.txt
```

#### Download All Available Files at once

```shell-session
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

Once we have downloaded all the files, `wget` will create a directory with the name of the IP address of our target. All downloaded files are stored there, which we can then inspect locally.

```shell-session
hellopriyanshu2702@htb[/htb]$ tree .
```

---
#### Upload a File

The ability to upload files to the FTP server connected to a web server increases the likelihood of gaining direct access to the webserver and even a reverse shell that allows us to execute internal system commands and perhaps even escalate our privileges.

```shell-session
hellopriyanshu2702@htb[/htb]$ touch testupload.txt
```

```shell-session
ftp> put testupload.txt 
```

---
## Footprinting the Service

```shell-session
Update nmap db
sudo nmap --script-updatedb
```

```shell-session
# All the Nmap scripts
find / -type f -name ftp* 2>/dev/null | grep scripts
```

```shell-session
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

```shell-session
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

# Nmap also provides the ability to trace the progress of NSE scripts at the network level if we use the `--script-trace` option in our scans.
```

---

we can, of course, use other applications such as `netcat` or `telnet` to interact with the FTP server.

```shell-session
nc -nv 10.129.14.136 21
```

```shell-session
telnet 10.129.14.136 21
```

It looks slightly different if the FTP server runs with TLS/SSL encryption. Because then we need a client that can handle TLS/SSL. For this, we can use the client `openssl` and communicate with the FTP server. The good thing about using `openssl` is that we can see the SSL certificate, which can also be helpful.

```shell-session
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

























 ---



```bash
# ls, get

┌─[eu-academy-6]─[10.10.14.38]─[htb-ac-1376914@htb-oazvtuvymn]─[~]
└──╼ [★]$ ftp 10.129.202.5
Connected to 10.129.202.5.
220 InFreight FTP v1.1
Name (10.129.202.5:root): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||24954|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftpuser  ftpuser        39 Nov  8  2021 flag.txt
226 Transfer complete
	ftp> get flag.txtmm  mm
	local: flag.txt remote: flag.txt
	229 Entering Extended Passive Mode (|||3730|)
	150 Opening BINARY mode data connection for flag.txt (39 bytes)
    39       31.89 KiB/s 
226 Transfer complete
39 bytes received in 00:00 (0.71 KiB/s)
ftp> exit
221 Goodbye.

┌─[eu-academy-6]─[10.10.14.38]─[htb-ac-1376914@htb-oazvtuvymn]─[~]
└──╼ [★]$ cat flag.txt
HTB{b7skjr4c76zhsds7fzhd4k3ujg7nhdjre}
┌─[eu-academy-6]─[10.10.14.38]─[htb-ac-1376914@htb-oazvtuvymn]─[~]
└──╼ [★]$ 
```

```bash
#NMAP

┌─[eu-academy-6]─[10.10.14.38]─[htb-ac-1376914@htb-oazvtuvymn]─[~]
└──╼ [★]$ sudo nmap -p21 -sV -sC -A 10.129.202.5
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-14 22:40 CST
Nmap scan report for 10.129.202.5
Host is up (0.052s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 InFreight FTP v1.1
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 InFreight FTP v1.1
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=12/14%Time=693F915F%P=x86_64-pc-linux-gnu%r(
SF:NULL,18,"220\x20InFreight\x20FTP\x20v1\.1\r\n")%r(GenericLines,74,"220\
SF:x20InFreight\x20FTP\x20v1\.1\r\n500\x20Invalid\x20command:\x20try\x20be
SF:ing\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20being\x
SF:20more\x20creative\r\n");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (95%), Linux 5.0 - 5.4 (95%), Linux 5.3 - 5.4 (95%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 2.6.32 (94%), Linux 5.0 (94%), Linux 5.0 - 5.5 (94%), HP P2000 G3 NAS device (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   52.00 ms 10.10.14.1
2   52.17 ms 10.129.202.5

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.75 seconds
```

