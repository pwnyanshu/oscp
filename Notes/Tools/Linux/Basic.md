![[Pasted image 20250819203145.png]]


| Command | Description             | Notes                                        |
| ------- | ----------------------- | -------------------------------------------- |
| ls      | listing                 | ls                                           |
| cat     | concatinate             | shows the content of a file                  |
| pwd     | print working directory |                                              |
| cd      | change directory        |                                              |
| find    |                         | find -name passwords.txt<br>find -name *.txt |
| grep    |                         | grep "81.143.211.90" access.log              |
| touch   | touch                   | Create file                                  |
| mkdir   | make directory          | Create a folder                              |
| cp      | copy                    | Copy a file or folder                        |
| mv      | move                    | Move a file or folder                        |
| rm      | remove                  | Remove a file or folder                      |
| file    | file                    | Determine the type of a file                 |
| su      |                         | sudo<br>su user2<br>su -l user2              |

| Operator | Meaning                               | Example                    |
| -------- | ------------------------------------- | -------------------------- |
| &        | Run command in the background         | `sleep 10 &`               |
| &&       | Logical AND (run next if successful)  | `mkdir dir && cd dir`      |
| >        | Redirect output to a file (overwrite) | `echo "Hello" > file.txt`  |
| >>       | Redirect output to a file (append)    | `echo "World" >> file.txt` |

# SSH (Secure Shell)
connect and control remote linux machine via terminal

![[Pasted image 20250814094229.png]]
its a protocol, it encrypts the data before transmission

ssh username@ip_address like ssh tryhackme@10.10.184.129

use nano, vim to edit files in ssh

| command | use                                                                              | notes                           |
| ------- | -------------------------------------------------------------------------------- | ------------------------------- |
| wget    | wget https://assets.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt | download files from web         |
| scp     | scp important.txt ubuntu@192.168.1.30:/home/ubuntu/transferred.txt               | copy from local to ssh computer |
| scp     | scp ubuntu@192.168.1.30:/home/ubuntu/documents.txt notes.txt                     | copy from ssh to local          |
# Make a ubuntu a webserver
python3 -m http.server
	where `-m` tells Python: “Find this module and execute its main entry point.”
![[Pasted image 20250815113623.png]]

![[Pasted image 20250815113640.png]]


# To see the running processes

| Command | notes    |
| ------- | -------- |
| ps      | rot user |
| ps aux  | all user |
| top     | dynamic  |

- SIGTERM - Kill the process, but allow it to do some cleanup tasks beforehand
- SIGKILL - Kill the process - doesn't do any cleanup after the fact
- SIGSTOP - Stop/suspend a process

![[Pasted image 20250815131613.png]]
![[Pasted image 20250815131627.png]]
![[Pasted image 20250815131638.png]]


## start stop a service

like in windows we used to do - net start apache24

in linux we do
systemctl start apache2 - just start it now
systemctl stop apache2
systemctl enable apache2 - configure to always start on boot
systemctl disable apache2


# Foregroung / background a process

background - ctrl + Z or & operator
foreground - fg


# cron jobs

# Syntax:
 ┌───────────── minute (0 - 59)
 │ ┌───────────── hour (0 - 23)
 │ │ ┌───────────── day of month (1 - 31)
 │ │ │ ┌───────────── month (1 - 12)
 │ │ │ │ ┌───────────── day of week (0 - 7) (Sun=0 or 7)
 │ │ │ │ │
 * * * * * <command>   # Redirect output: <command> >> /path/to/log 2>&1

 Special strings:
 @reboot    → Run once at startup
 @yearly    → 0 0 1 1 *
 @monthly   → 0 0 1 * *
 @weekly    → 0 0 * * 0
 @daily     → 0 0 * * *
 @hourly    → 0 * * * *

 Example: Run script at midnight every day
0 0 * * * /path/to/script.sh

 Example: Run every 5 minutes
*/5 * * * * /path/to/script.sh

 Example: Backup every 12 hours (midnight & noon)
0 */12 * * * cp -R /home/user/Documents /var/backups/

 Example: Run script on boot
@reboot /path/to/script.sh

# add apt repository

![[Pasted image 20250815154501.png]]

