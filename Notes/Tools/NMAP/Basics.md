## In tcp scan -sT,
this happens for open ports
![[Pasted image 20250831222846.png]]

and this for closed ports
![[Pasted image 20250831222925.png]]

## In SYN Scan -sS
instead of ACK we send RST, it is considered to some what stealth
![[Pasted image 20250831223106.png]]

## NULL, FIN and Xmas 
are all considered stealth

Null
![[Pasted image 20250831223215.png]]

FIN
![[Pasted image 20250831223225.png]]


Xmas
![[Pasted image 20250831223238.png]]


## NSE Scripts
can be searched like 
grep "ftp" /usr/share/nmap/scripts/script.db
ls -l /usr/share/nmap/scripts/*ftp*


