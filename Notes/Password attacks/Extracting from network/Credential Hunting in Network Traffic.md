Most applications use TLS to encrypt data in transit. 

However legacy systems, misconfigured services, or test applications launched without HTTPS can still result in the use of unencrypted protocols such as HTTP or SNMP.

These gaps present a valuable opportunity for attackers: the chance to hunt for credentials in cleartext network traffic.

|Unencrypted Protocol|Encrypted Counterpart|Description|
|---|---|---|
|`HTTP`|`HTTPS`|Used for transferring web pages and resources over the internet.|
|`FTP`|`FTPS/SFTP`|Used for transferring files between a client and a server.|
|`SNMP`|`SNMPv3 (with encryption)`|Used for monitoring and managing network devices like routers and switches.|
|`POP3`|`POP3S`|Retrieves emails from a mail server to a local client.|
|`IMAP`|`IMAPS`|Accesses and manages email messages directly on the mail server.|
|`SMTP`|`SMTPS`|Sends email messages from client to server or between mail servers.|
|`LDAP`|`LDAPS`|Queries and modifies directory services like user credentials and roles.|
|`RDP`|`RDP (with TLS)`|Provides remote desktop access to Windows systems.|
|`DNS (Traditional)`|`DNS over HTTPS (DoH)`|Resolves domain names into IP addresses.|
|`SMB`|`SMB over TLS (SMB 3.0)`|Shares files, printers, and other resources over a network.|
|`VNC`|`VNC with TLS/SSL`|Allows graphical remote control of another computer.|

---
# Wireshark

It features a powerful [filter engine](https://www.wireshark.org/docs/man-pages/wireshark-filter.html) that allows for efficient searching through both live and captured network traffic. Some basic but useful filters include:

|Wireshark filter|Description|
|---|---|
|`ip.addr == 56.48.210.13`|Filters packets with a specific IP address|
|`tcp.port == 80`|Filters packets by port (HTTP in this case).|
|`http`|Filters for HTTP traffic.|
|`dns`|Filters DNS traffic, which is useful to monitor domain name resolution.|
|`tcp.flags.syn == 1 && tcp.flags.ack == 0`|Filters SYN packets (used in TCP handshakes), useful for detecting scanning or connection attempts.|
|`icmp`|Filters ICMP packets (used for Ping), which can be useful for reconnaissance or network issues.|
|`http.request.method == "POST"`|Filters for HTTP POST requests. In the case that POST requests are sent over unencrypted HTTP, it may be the case that passwords or other sensitive information is contained within.|
|`tcp.stream eq 53`|Filters for a specific TCP stream. Helps track a conversation between two hosts.|
|`eth.addr == 00:11:22:33:44:55`|Filters packets from/to a specific MAC address.|
|`ip.src == 192.168.24.3 && ip.dst == 56.48.210.3`|Filters traffic between two specific IP addresses. Helps track communication between specific hosts.|

Find HTTP traffic
![[Pasted image 20260317201947.png]]

Find certain string
We can do either (http contains "passw") 
or
`Edit > Find Packet` and enter the desired search query manually.

![[Pasted image 20260317202217.png]]

![[Pasted image 20260323183346.png]]

We should use RegEx too

---

# Pcredz

[Pcredz](https://github.com/lgandx/PCredz) is a tool that can be used to extract credentials from live traffic or network packet captures. Specifically, it supports extracting the following information:

- Credit card numbers
- POP credentials
- SMTP credentials
- IMAP credentials
- SNMP community strings
- FTP credentials
- Credentials from HTTP NTLM/Basic headers, as well as HTTP Forms
- NTLMv1/v2 hashes from various traffic including DCE-RPC, SMBv1/2, LDAP, MSSQL, and HTTP
- Kerberos (AS-REQ Pre-Auth etype 23) hashes

In order to run `Pcredz`, one may either clone the repository and install all dependencies, or use the provided Docker container detailed in the [Install](https://github.com/lgandx/PCredz?tab=readme-ov-file#install) portion of the README file.

The following command can be used to run `Pcredz` against a packet capture file:

```bash
./Pcredz -f demo.pcapng -t -v
```

I used docker

See how to install it over here - [[Install Docker and Pcredz]]

![[Pasted image 20260323183431.png]]