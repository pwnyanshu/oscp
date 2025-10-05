
### One-line definitions

- **Lateral movement:** after you own Host A, you move _sideways_ to other hosts on the same network (Host B, C…) to expand access.
    
- **Pivoting:** you use a compromised host as a **jump point** to reach network segments or hosts that your original machine cannot see.
    
- **Tunneling:** you create a **pipe** (SSH/SOCKS/HTTP/etc.) that forwards traffic through one host to another — the technical mechanism that enables pivoting.
    

### Quick differences (ruthless clarity)

- Lateral movement = _what you do_ (technique/goal: compromise more machines).
    
- Pivoting = _how you gain access to otherwise unreachable networks_ (use Host A to reach Hosts behind it).
    
- Tunneling = _the plumbing_ (SSH tunnels, SOCKS, socat, reverse tunnels) that makes pivoting and some lateral moves possible.