

**Netcat** (usually `nc`) is a **raw TCP/UDP pipe**.

It just **reads bytes from one place and writes them to another**.

Server
```shell-session
nc -lvnp 7777

-l → listen (act as server)
-v → verbose
-n → no DNS lookup
-p 7777 → listen on port 7777
```

Client
```shell-session
nc -nv 10.129.41.200 7777

-v → verbose
-n → no DNS lookup
```

