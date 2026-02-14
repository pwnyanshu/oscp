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

As Splunk can be installed on Windows or Linux hosts, scripted inputs can be created to run Bash, PowerShell, or Batch scripts. Also, every Splunk installation comes with Python installed, so Python scripts can be run on any Splunk system. A quick way to gain RCE is by creating a scripted input that tells Splunk to run a Python reverse shell script. We'll cover this in the next section.

Aside from this built-in functionality, Splunk has suffered from various public vulnerabilities over the years, such as this [SSRF](https://www.exploit-db.com/exploits/40895) that could be used to gain unauthorized access to the Splunk REST API. At the time of writing, Splunk has [47](https://www.cvedetails.com/vulnerability-list/vendor_id-10963/Splunk.html) CVEs. If we perform a vulnerability scan against Splunk during a penetration test, we will often see many non-exploitable vulnerabilities returned. This is why it is important to understand how to abuse built-in functionality.

![[Pasted image 20260128160027.png]]

---

# Attacking Splunk

we can gain remote code execution on Splunk by creating a custom application to run Python, Batch, Bash, or PowerShell scripts.

## Abusing Built-In Functionality

We can use [this](https://github.com/0xjpuff/reverse_shell_splunk) Splunk package to assist us. The bin directory in this repo has examples for [Python](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/rev.py) and [PowerShell](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/run.ps1).**
