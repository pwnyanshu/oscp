
Network file system (SMB for Linux server)

Who uses NFS?
Serious Unix/Linux infrastructure still uses NFS, because it solves a very specific problem extremely well.

### A brutal analogy (worth keeping)

- WhatsApp / Gmail → mailing documents to coworkers
- FTP → courier service
- NFS / SMB → shared office filing cabinet **that everyone uses at the same time**
- Object storage (S3) → giant archive, not a filesystem

Its purpose is to access file systems over a network as if they were local. However, it uses an entirely different protocol. [NFS](https://en.wikipedia.org/wiki/Network_File_System) is used between Linux and Unix systems.

|**Version**|**Features**|
|---|---|
|`NFSv2`|It is older but is supported by many systems and was initially operated entirely over UDP.|
|`NFSv3`|It has more features, including variable file size and better error reporting, but is not fully compatible with NFSv2 clients.|
|`NFSv4`|It includes Kerberos, works through firewalls and on the Internet, no longer requires portmappers, supports ACLs, applies state-based operations, and provides performance improvements and high security. It is also the first version to have a stateful protocol.|
### Why NFS is dangerous (this is the key insight)

NFS **trusts the client** to tell the truth about who it is.

Let that sink in.

When a client says:

> “Hi, I’m UID 0 (root)”

The server usually replies:

> “Cool, root it is.”

Unless explicitly configured otherwise.

That’s insane by modern security standards—and that’s why NFS is supposed to live only inside trusted networks.

---

## Authentication vs Authorization 

Let’s separate them clearly:

### Authentication

> “Who are you?”

Classic NFS (v2/v3):

- **No real authentication**
- RPC just passes UID/GID numbers

NFSv4:

- Can use Kerberos
- This is the _only_ version that behaves like a modern system

### Authorization

> “What can you access?”

Based purely on:

- File permissions
- UID/GID matching

If your UID matches the file owner → you get access  
No second opinion. No extra checks.

---

## Why UID/GID mismatch is a vulnerability

Example:

Server file:

`-rw-r--r--  root root  id_rsa`

That’s UID 0.

If the client:

- Creates a local user with UID 0
- Or mounts without root squashing

Boom.  
You are effectively root _on the share_.

This is why attackers love misconfigured NFS.

---

### `/etc/exports` — the control center ( Default Configuration )

This file answers three questions:

1. **What folder is shared**
2. **Who can access it**
3. **How much power they get**

Example:

`/mnt/nfs 10.129.14.0/24(rw,sync,no_root_squash)`

Translation:

- Share `/mnt/nfs`
- Anyone in that subnet
- Can read & write
- Root stays root
That last line is the disaster switch.

```shell-session
cat /etc/exports 
```

The default `exports` file also contains some examples of configuring NFS shares. First, the folder is specified and made available to others, and then the rights they will have on this NFS share are connected to a host or a subnet. Finally, additional options can be added to the hosts or subnets.

|**Option**|**Description**|
|---|---|
|`rw`|Read and write permissions.|
|`ro`|Read only permissions.|
|`sync`|Synchronous data transfer. (A bit slower)|
|`async`|Asynchronous data transfer. (A bit faster)|
|`secure`|Ports above 1024 will not be used.|
|`insecure`|Ports above 1024 will be used.|
|`no_subtree_check`|This option disables the checking of subdirectory trees.|
|`root_squash`|Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount.|

Let us create such an entry for test purposes and play around with the settings.

#### ExportFS

  NFS

```shell-session
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server 
root@nfs:~# exportfs

/mnt/nfs      	10.129.14.0/24
```

We have shared the folder `/mnt/nfs` to the subnet `10.129.14.0/24` with the setting shown above. This means that all hosts on the network will be able to mount this NFS share and inspect the contents of this folder.

---

## Dangerous Settings

However, even with NFS, some settings can be dangerous for the company and its infrastructure. Here are some of them listed:

| **Option**       | **Description**                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `rw`             | Read and write permissions.                                                                                          |
| `insecure`       | Ports above 1024 will be used.                                                                                       |
| `nohide`         | If another file system was mounted below an exported directory, this directory is exported by its own exports entry. |
| `no_root_squash` | All files created by root are kept with the UID/GID 0.                                                               |

---

### Root squash (this is non-negotiable to understand)

#### With `root_squash` (safe-ish)

- Client root (UID 0) → mapped to UID 65534 (`nobody`)
- Root loses power

#### With `no_root_squash` (game over)

- Client root = server root
- You can:
    - Drop SUID binaries
    - Modify scripts
    - Steal SSH keys
    - Escalate privileges later

If you don’t immediately flinch when you see `no_root_squash`, you haven’t understood NFS yet.

---
## Footprinting NFS

### Ports — why 111 and 2049 matter

- **111** → RPC binder  
    “What services exist on this machine?”
    
- **2049** → NFS itself

```shell-session
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
```

![[Pasted image 20260129125245.png]]

The `rpcinfo` NSE script retrieves a list of all currently running RPC services, their names and descriptions, and the ports they use.

This lets us check whether the target share is connected to the network on all required ports. Also, for NFS, Nmap has some NSE scripts that can be used for the scans. These can then show us, for example, the `contents` of the share and its `stats`.

```shell-session
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:37 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0      0      0     2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount: 
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs: 
|   Filesystem  1K-blocks   Used       Available   Use%  Maxfilesize  Maxlink
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%   16.0T        32000
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41982/udp6  mountd
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      47217/tcp6  mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      39542/udp   nlockmgr
|   100021  1,3,4      44629/tcp   nlockmgr
|   100021  1,3,4      45273/tcp6  nlockmgr
|   100021  1,3,4      47524/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```

Once we have discovered such an NFS service, we can mount it on our local machine. For this, we can create a new empty folder to which the NFS share will be mounted. Once mounted, we can navigate it and view the contents just like our local system.

#### Show Available NFS Shares

  NFS

```shell-session
hellopriyanshu2702@htb[/htb]$ showmount -e 10.129.14.128

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24


-e: stands for exports.
```

![[Pasted image 20260129125458.png]]

#### Mounting NFS Share

  NFS

```shell-session
hellopriyanshu2702@htb[/htb]$ mkdir target-NFS
hellopriyanshu2702@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
hellopriyanshu2702@htb[/htb]$ cd target-NFS
hellopriyanshu2702@htb[/htb]$ tree .

.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files



-t nfs — filesystem type - This filesystem speaks the NFS protocol
10.129.14.128:/ - <server_ip>:<export_root>
-o nolock - Do not use file locking. Just let me read/write
```

There we will have the opportunity to access the rights and the usernames and groups to whom the shown and viewable files belong. Because once we have the usernames, group names, UIDs, and GUIDs, we can create them on our system and adapt them to the NFS share to view and modify the files.

#### List Contents with Usernames & Group Names

  NFS

```shell-session
hellopriyanshu2702@htb[/htb]$ ls -l mnt/nfs/

total 16
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 cry0l1t3 cry0l1t3  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1 root     root        0 Sep 19 17:22 nfs.share
```

#### List Contents with UIDs & GUIDs

  NFS

```shell-session
hellopriyanshu2702@htb[/htb]$ ls -n mnt/nfs/

total 16
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 1000 1000  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1    0 1000 1221 Sep 19 18:21 backup.sh
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share
```

It is important to note that if the `root_squash` option is set, we cannot edit the `backup.sh` file even as `root`.
With `root_squash` (safe-ish)

- Client root (UID 0) → mapped to UID 65534 (`nobody`)
- Root loses power

 With `no_root_squash` (game over)

- Client root = server root
- You can:
    - Drop SUID binaries
    - Modify scripts
    - Steal SSH keys
    - Escalate privileges later

We can also use NFS for further escalation. For example, if we have access to the system via SSH and want to read files from another folder that a specific user can read, we would need to upload a shell to the NFS share that has the `SUID` of that user and then run the shell via the SSH user.

After we have done all the necessary steps and obtained the information we need, we can unmount the NFS share.

#### Unmounting

  NFS

```shell-session
hellopriyanshu2702@htb[/htb]$ cd ..
hellopriyanshu2702@htb[/htb]$ sudo umount ./target-NFS
```


![[Pasted image 20260129125902.png]]
