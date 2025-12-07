
Why do we want to hack these
- user credentials
- Personal Identifiable Information (PII)
- business-related data
- payment information
- lateral movement and privilege escalation
---

---

## Enumeration

- MSSQL - `TCP/1433` and `UDP/1434`, 
- MySQL - `TCP/3306`
- MSSQL operates in a "hidden" mode - `TCP/2433`

---

```shell-session
nmap -Pn -sV -sC -p1433 10.10.10.125
```

![[Pasted image 20251105231451.png]]

