In many cases, `symmetric encryption` algorithms such as `AES-256` are used to securely store individual files or folders. In this method, the same key is used for both encryption and decryption. 

For transmitting files, `asymmetric encryption` is typically employed, which uses two distinct keys: the sender encrypts the file with the recipient's `public key`, and the recipient decrypts it using the corresponding `private ke`y.

---
## Finding files which are mostly password protected

```shell
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

---
## ## Cracking encrypted SSH keys

Finding SSH Keys
```shell
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

Some ssh keys are themselves encrypted, 
SO how do we tell if its encrypted or not, because they all look exactly same

```shell
ssh-keygen -yf ~/.ssh/id_ed25519

# if it asks for password then it is not encrypted, else its encrypted
```

Lets start Cracking encrypted SSH keys

```shell
# Find where is ssh2john
locate *2john* 

# Convert it to johns format
ssh2john.py SSH.private > ssh.hash

# cracking started
john --wordlist=rockyou.txt ssh.hash

# show the cracked password
john ssh.hash --show 
```

---
## Cracking password-protected documents

```shell
office2john.py Protected.docx > protected-docx.hash 
john --wordlist=rockyou.txt protected-docx.hash
john protected-docx.hash --show
```
![[Pasted image 20260308015139.png]]
![[Pasted image 20260308015216.png]]
![[Pasted image 20260308015237.png]]

```shell
pdf2john.py PDF.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
john pdf.hash --show
```

Better the quality of password, better the chanced to crack the files
