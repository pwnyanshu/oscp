
### One-line definitions

- **Lateral movement:** after you own Host A, you move _sideways_ to other hosts on the same network (Host B, C…) to expand access.
    
- **Pivoting:** you use a compromised host as a **jump point** to reach network segments or hosts that your original machine cannot see.
    
- **Tunneling:** you create a **pipe** (SSH/SOCKS/HTTP/etc.) that forwards traffic through one host to another — the technical mechanism that enables pivoting.
    

### Quick differences (ruthless clarity)

- Lateral movement = _what you do_ (technique/goal: compromise more machines).
    
- Pivoting = _how you gain access to otherwise unreachable networks_ (use Host A to reach Hosts behind it).
    
- Tunneling = _the plumbing_ (SSH tunnels, SOCKS, socat, reverse tunnels) that makes pivoting and some lateral moves possible.

---

**IP**
ifconfig / ipconfig

**Routing table**
netstat -r / ip route


---

## Pivoting, Tunneling & Routing — Core Networking Notes

### 1. IP Address & NIC Basics

- Every device on a network must have an **IP address**.
- An IP is always assigned to a **NIC (Network Interface Controller)**.
- A system can have **multiple NICs**, physical or virtual.
- Each NIC can connect the host to a **different network**.

Common NICs you’ll see:

- `eth0` → Public / Internet-facing interface
- `eth1` → Internal / private network
- `tun0` → VPN tunnel interface
- `lo` → Loopback (127.0.0.1)

Public vs Private IPs:

- **Public IPs** are routable on the Internet.
- **Private IPs** (10.x, 172.16–31.x, 192.168.x) are routable only inside internal networks.
- NAT translates private IPs to public IPs for Internet access.

---

### 2. VPN & tun Interfaces

- When a VPN connects, a **tunnel interface (`tun0`)** is created.
- `tun0` represents an **encrypted tunnel** into another network.
- Networks behind the VPN are **unreachable without this tunnel**.
- VPN does NOT automatically route all traffic — only networks explicitly listed in routes.

---

### 3. Routing Table Fundamentals

- Every OS has a **routing table**.
- Routing decisions are made **before packets leave the machine**.
- The routing table decides:
    - **Which gateway**
    - **Which NIC**
    - **Which path** traffic takes

Key rule:

> **Longest Prefix Match always wins**

Meaning:

- The most specific subnet mask (largest /CIDR) is chosen.
- Interfaces do not matter unless the route matches.

---

### 4. Default Gateway (Gateway of Last Resort)

- If **no route matches** the destination IP:
    
    - Traffic is sent to the **default route**
        
- The default gateway usually leads to:
    
    - The Internet
        
    - An upstream router
        

Example:

`default → 178.62.64.1 → eth0`

Any public website (e.g. www.hackthebox.com):

- Does NOT match private routes
    
- Goes to the **default gateway**
    
- Exits via **eth0**
    

---

### 5. Example Routing Decisions (From Pwnbox)

Routing table entry:

`10.129.0.0/16 → tun0 10.106.0.0/20 → eth1 default        → eth0`

Case 1: Destination `10.129.10.25`

- Matches `10.129.0.0/16`
    
- Sent via **tun0**
    
- Reason: VPN route + longest prefix match
    

Case 2: Destination `www.hackthebox.com`

- Public IP
    
- Matches no internal routes
    
- Sent to **default gateway (178.62.64.1)**
    
- Exits via **eth0**
    

---

### 6. Why Routing Matters for Pivoting

- A compromised host can only reach:
    
    - Networks listed in its routing table
        
- Pivoting means:
    
    - Using that host to **route traffic into networks you can’t reach directly**
        
- Always inspect:
    
    - `ifconfig` / `ipconfig`
        
    - `netstat -r` / `ip route`
        

If you don’t understand the routing table:

- You’re guessing
    
- Your tunnels will fail
    
- Your pivot will be blind
    

---

### 7. Mental Model (Important)

- **Interfaces do nothing by themselves**
    
- **Routes make decisions**
    
- VPNs are not magic
    
- Traffic goes **only where routes allow**
    

Pivoting is not hacking magic —  
it is **controlled packet steering**.

