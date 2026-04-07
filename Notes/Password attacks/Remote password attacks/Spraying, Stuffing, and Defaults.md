
---
## Password spraying

- brute-force attack in which an attacker attempts to use a single password across many different user accounts
- effective in environments where users are initialized with a default or standard password
- For example, if it is known that administrators at a particular company commonly use `ChangeMe123!` when setting up new accounts, it would be worthwhile to spray this password across all user accounts to identify any that were not updated.

web applications - [Burp Suite](https://portswigger.net/burp) 
Active Directory environments - [NetExec](https://github.com/Pennyw0rth/NetExec) or [Kerbrute](https://github.com/ropnop/kerbrute)

```shell
netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```

---
## Credential stuffing

- another type of brute-force attack in which an attacker uses stolen credentials from one service to attempt access on others
- Since many users reuse their usernames and passwords across multiple platforms (such as email, social media, and enterprise systems), these attacks are sometimes successful. As with password spraying, credential stuffing can be carried out using a variety of tools, depending on the target system
- if we have a list of `username:password` credentials obtained from a database leak, we can use `hydra` to perform a credential stuffing attack against an SSH service using the following syntax

```bash
hydra -C user_pass.list ssh://10.100.38.23
```

---
## Default credentials

- Many systems—such as routers, firewalls, and databases—come with `default credentials`
- While several lists of known default credentials are available online, there are also dedicated tools that automate the process. One widely used example is the [Default Credentials Cheat Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet), which we can install with `pip3`.

```shell
pip3 install defaultcreds-cheat-sheet
```

- Once installed, we can use the `creds` command to search for known default credentials associated with a specific product or vendor.

![[Pasted image 20260405114056.png]]

- In addition to publicly available lists and tools, default credentials can often be found in product documentation

- Let's imagine we have identified certain applications in use on a customer's network. After researching the default credentials online, we can combine them into a new list, formatted as `username:password`, and reuse the previously mentioned `hydra` command to attempt access.

- Beyond applications, default credentials are also commonly associated with routers. One such list is available [here](https://www.softwaretestinghelp.com/default-router-username-and-password-list/).

|**Router Brand**|**Default IP Address**|**Default Username**|**Default Password**|
|---|---|---|---|
|3Com|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|Belkin|[http://192.168.2.1](http://192.168.2.1/)|admin|admin|
|BenQ|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|D-Link|[http://192.168.0.1](http://192.168.0.1/)|admin|Admin|
|Digicom|[http://192.168.1.254](http://192.168.1.254/)|admin|Michelangelo|
|Linksys|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|Netgear|[http://192.168.0.1](http://192.168.0.1/)|admin|password|

Got access to MySQL using defailt found in the cheat sheet mentioned above.
![[Pasted image 20260405131803.png]]


---

