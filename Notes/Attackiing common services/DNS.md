
Ports:
- `UDP/53` - currently
- `TCP/53` - more heavily in future

---
## Enumeration

We can understand how a company operates and the services they provide, as well as third-party service providers like emails.

```bash
nmap -p53 -Pn -sV -sC 10.10.110.213
```

