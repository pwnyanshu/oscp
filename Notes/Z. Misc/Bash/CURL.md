
## GET

curl -sOJ http://94.237.123.185:36776/download.php?contract="$hash"
- `-s` (silent)  
    Hides progress _and_ errors. That’s fine for scripts, **bad for debugging**. If this fails, you’ll be blind.
- `-O`  
    Saves the response to a file using the **remote filename**.
- `-J`  
    Tells curl to **trust `Content-Disposition` headers** to decide the filename

---
## POST

```
```
```
```
```
```
```

curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php

- `-s` (silent)  
    Hides progress _and_ errors. That’s fine for scripts, **bad for debugging**. If this fails, you’ll be blind.
- `-O`  
    Saves the response to a file using the **remote filename**.
- `-J`  
    Tells curl to **trust `Content-Disposition` headers** to decide the filename
    `-X` tells curl **which HTTP verb to use**.
    -d Send data in request body

```