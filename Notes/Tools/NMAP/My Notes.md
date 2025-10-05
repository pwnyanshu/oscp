```bash

sudo nmap -sT 10.10.10.10 #TCP connect scan
-F Fast only 100 ports
-r in asc order
-vv very verbose # IMP use always
-sV also shows service version
-sC runs basic nsc scripts
-p- all 65K ports
-p10-20, -p20,80,443
-T0 slowest, -T1 real life, -T3 Default, -T4 in CTF events, -T5 fastest 
--max-rate 50 rate <= 50 packets/sec
--min-rate 15 rate >= 15 packets/sec
--min-parallelism 100 at least 100 probes in parallel 

sudo nmap -sU 10.10.10.10 #UDP Scan
# If recieve no reply it states them as open
# If recieves ICMP 3 type 3 then stats them as close

sudo nmap -sS 10.10.10.10 # TCP Sync scan
# only works if we are a admin or sudoer
# its the default if we are admin, else -sT is default




sudo nmap -p 80 --script=all -sV -Pn -v 10.10.10.10


```


## Trying to fool the firewall
```bash

SUDO nmap -sN 10.10.10.10 # Null Scan
# sets no flag
# Open - recieves nothing
# close - recieves rst, ack

SUDO nmap -sF # FIN scan
# sets finish flag
# Open - recieves nothing
# close - recieves rst, ack

SUDO nmap -sX # x-mas scan
# sets fin,psh,urg flag
# Open - recieves nothing
# close - recieves rst, ack

```

