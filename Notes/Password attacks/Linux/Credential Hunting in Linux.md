
Hunting for credentials is one of the first steps once we have access to the system.

These low-hanging fruits can give us elevated privileges within seconds or minutes.

Lets say he have access for the local user. To escalate our privileges most efficiently, we can search for passwords or even whole credentials that we can use to log in to our target.

---

There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

- `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
- `History` including logs, and command-line history
- `Memory` including cache, and in-memory processing
- `Key-rings` such as browser stored credentials

---
---

# Files

One core principle of Linux is that everything is a file.

We should look for, find, and inspect several categories of files one by one.
Like
- Configuration files
- Databases
- Notes
- Scripts
- Cronjobs
- SSH keys

---
### Searching for configuration files

Configuration files are the core of the functionality of services on Linux distributions. Often they even contain credentials that we will be able to read.

**Usually, the configuration files are marked with the following three file extensions (`.config`, `.conf`, `.cnf`).**

 However, these configuration files or the associated extension files can be renamed, which means that these file extensions are not necessarily required, however rare but this possibility should not be left out of our search.

```bash
# Search all configuration files
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

We can save the output in text and check it one after the another OR

run the scan directly for each file found with the specified file extension and output the contents. In this example, we search for three words (`user`, `password`, `pass`) in each file with the file extension `.cnf`

```bash
# search words like user password pass in all .cnf files
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```


---
### Searching for databases

```bash

for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

---
### Searching for notes

Depending on the environment we are in and the purpose of the host we are on, we can often find notes about specific processes on the system. These often include lists of many different access points or even their credentials.

The notes can have .txt extension or even can be extension less.

```bash
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

---
### Searching for scripts
Scripts are files that often contain highly sensitive information and processes. Among other things, these also contain credentials that are necessary to be able to call up and execute the processes automatically. Otherwise, the administrator or developer would have to enter the corresponding password each time the script or the compiled program is called.


```bash
hellopriyanshu2702@htb[/htb]$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

---
### Enumerating cronjobs

`/etc/crontab`

Some applications and scripts require credentials to run and are therefore incorrectly entered in the cronjobs. 

Furthermore, there are the areas that are divided into different time ranges (`/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.monthly`, `/etc/cron.weekly`). 

The scripts and files used by `cron` can also be found in `/etc/cron.d/` for Debian-based distributions.

```bash
cat /etc/crontab
ls -la /etc/cron.*/
```

---
### Enumerating history files

We are interested in the files that store users' command history and the logs that store information about system processes.

In the history of the commands entered on Linux distributions that use Bash as a standard shell, we find the associated files in `.bash_history`. Nevertheless, other files like `.bashrc` or `.bash_profile` can contain important information.

```bash
tail -n5 /home/*/.bash*
```

---
### Enumerating log files

An essential concept of Linux systems is log files that are stored in text files.

The entirety of log files can be divided into four categories:

- Application logs
- Event logs
- Service logs
- System logs

Many different logs exist on the system. These can vary depending on the applications installed, but here are some of the most important ones:

|**File**|**Description**|
|---|---|
|`/var/log/messages`|Generic system activity logs.|
|`/var/log/syslog`|Generic system activity logs.|
|`/var/log/auth.log`|(Debian) All authentication related logs.|
|`/var/log/secure`|(RedHat/CentOS) All authentication related logs.|
|`/var/log/boot.log`|Booting information.|
|`/var/log/dmesg`|Hardware and drivers related information and logs.|
|`/var/log/kern.log`|Kernel related warnings, errors and logs.|
|`/var/log/faillog`|Failed login attempts.|
|`/var/log/cron`|Information related to cron jobs.|
|`/var/log/mail.log`|All mail server related logs.|
|`/var/log/httpd`|All Apache related logs.|
|`/var/log/mysqld.log`|All MySQL server related logs.|

 **We should familiarize ourselves with the individual logs, first examining them manually and understanding their formats.**

However, here are some strings we can use to find interesting content in the logs:

```bash
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

# Memory and cache

### Mimipenguin

Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused. For example, it may be the system-required credentials for the logged-in users. Another example is the credentials stored in the browsers, which can also be read. In order to retrieve this type of information from Linux distributions, there is a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin) that makes the whole process easier. However, this tool requires administrator/root permissions.

```bash
sudo python3 mimipenguin.py
```

### LaZagne

An even more powerful tool we can use that was mentioned earlier in the Credential Hunting in Windows section is `LaZagne`. This tool allows us to access far more resources and extract the credentials. The passwords and hashes we can obtain come from the following sources but are not limited to:

- Wifi
- Wpa_supplicant
- Libsecret
- Kwallet
- Chromium-based
- CLI
- Mozilla
- Thunderbird
- Git
- ENV variables
- Grub
- Fstab
- AWS
- Filezilla
- Gftp
- SSH
- Apache
- Shadow
- Docker
- Keepass
- Mimipy
- Sessions
- Keyrings

```bash
sudo python2.7 laZagne.py all
```

# Browser credentials

Browsers store the passwords saved by the user in an encrypted form locally on the system to be reused. For example, the `Mozilla Firefox` browser stores the credentials encrypted in a hidden folder for the respective user. These often include the associated field names, URLs, and other valuable information.

For example, when we store credentials for a web page in the Firefox browser, they are encrypted and stored in `logins.json` on the system. However, this does not mean that they are safe there. Many employees store such login data in their browser without suspecting that it can easily be decrypted and used against the company.

```bash
ls -l .mozilla/firefox/ | grep default

cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```

The tool [Firefox Decrypt](https://github.com/unode/firefox_decrypt) is excellent for decrypting these credentials, and is updated regularly. It requires Python 3.9 to run the latest version. Otherwise, `Firefox Decrypt 0.7.0` with Python 2 must be used.

```shell
python3.9 firefox_decrypt.py
```

Alternatively, `LaZagne` can also return results if the user has used the supported browser.

```bash
python3 laZagne.py browsers
```

