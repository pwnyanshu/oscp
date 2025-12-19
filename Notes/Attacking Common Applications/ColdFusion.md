ColdFusion is:

Basically websites are build using this

- a **Java-based application server**
- that serves **web apps written in CFML**
- usually sitting **behind or alongside a web server**

Code is similar to HTML
like
```html
<cfquery name="myQuery" datasource="myDataSource">
  SELECT *
  FROM myTable
</cfquery>
```

```html
<cfloop query="myQuery">
  <p>#myQuery.firstName# #myQuery.lastName#</p>
</cfloop>
```

---
Here are a few known vulnerabilities of ColdFusion:

1. CVE-2021-21087: Arbitrary disallow of uploading JSP source code
2. CVE-2020-24453: Active Directory integration misconfiguration
3. CVE-2020-24450: Command injection vulnerability
4. CVE-2020-24449: Arbitrary file reading vulnerability
5. CVE-2019-15909: Cross-Site Scripting (XSS) Vulnerability

---
ColdFusion exposes a fair few ports by default:

|Port Number|Protocol|Description|
|---|---|---|
|80|HTTP|Used for non-secure HTTP communication between the web server and web browser.|
|443|HTTPS|Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.|
|1935|RPC|Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device.|
|25|SMTP|Simple Mail Transfer Protocol (SMTP) is used for sending email messages.|
|8500|SSL|Used for server communication via Secure Socket Layer (SSL).|
|5500|Server Monitor|Used for remote administration of the ColdFusion server.|

---
## Enumeration

|**Method**|**Description**|
|---|---|
|`Port Scanning`|ColdFusion typically uses port 80 for HTTP and port 443 for HTTPS by default. So, scanning for these ports may indicate the presence of a ColdFusion server. Nmap might be able to identify ColdFusion during a services scan specifically.|
|`File Extensions`|ColdFusion pages typically use ".cfm" or ".cfc" file extensions. If you find pages with these file extensions, it could be an indicator that the application is using ColdFusion.|
|`HTTP Headers`|Check the HTTP response headers of the web application. ColdFusion typically sets specific headers, such as "Server: ColdFusion" or "X-Powered-By: ColdFusion", that can help identify the technology being used.|
|`Error Messages`|If the application uses ColdFusion and there are errors, the error messages may contain references to ColdFusion-specific tags or functions.|
|`Default Files`|ColdFusion creates several default files during installation, such as "admin.cfm" or "CFIDE/administrator/index.cfm". Finding these files on the web server may indicate that the web application runs on ColdFusion.|

```shell-session
nmap -p- -sC -Pn 10.129.247.30 --open
```

`8500` is a default port that ColdFusion uses for SSL. Navigating to the `IP:8500` lists 2 directories, `CFIDE` and `cfdocs,` in the root, further indicating that ColdFusion is running on port 8500.

---
