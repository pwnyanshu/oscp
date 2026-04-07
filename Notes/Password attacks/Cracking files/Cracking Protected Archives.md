
There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.

A comprehensive list of archive file types can be found on [FileInfo](https://fileinfo.com/filetypes/compressed). Rather than typing them out manually, we can also query the data using a one-liner, apply filters as needed, and save the results to a file.

List of all the compressed file types
```shell
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

Note that not all archive types support native password protection, and in such cases, additional tools are often used to encrypt the files. For example, TAR files are commonly encrypted using `openssl` or `gpg`.

---
## Cracking ZIP files

```shell
zip2john ZIP.zip > zip.hash

john --wordlist=rockyou.txt zip.hash

john zip.hash --show
```
---
## Cracking OpenSSL encrypted GZIP files

`openssl` can be used to encrypt files in the `GZIP` format. To determine the actual format of a file, we can use the `file` command, which provides detailed information about its contents. For example:

```shell
file GZIP.gzip  

# GZIP.gzip: openssl enc'd data with salted password
```

The following one-liner may produce several GZIP-related error messages, which can be safely ignored. If the correct password list is used, as in this example, we will see another file successfully extracted from the archive.

```shell
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

Once the `for` loop has finished, we can check the current directory for a newly extracted file.

---
## Cracking BitLocker-encrypted drives

- [BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10) is a full-disk encryption feature developed by Microsoft for the Windows OS
- it uses the `AES` encryption algorithm with either 128-bit or 256-bit key lengths
- If the password or PIN used for BitLocker is forgotten, decryption can still be performed using a recovery key—a 48-digit string generated during the setup process.

To crack a BitLocker encrypted drive, we can use a script called `bitlocker2john`

```shell
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash
```

![[Pasted image 20260404131946.png]]
![[Pasted image 20260404132015.png]]


Once a hash is generated, either `JtR` or `hashcat` can be used to crack it.

The hashcat mode associated with the `$bitlocker$0$...` hash is `-m 22100`

Since this encryption uses strong AES encryption, cracking may take considerable time depending on hardware performance.

```bash
hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt

hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt --show
```
![[Pasted image 20260404132055.png]]
![[Pasted image 20260404132110.png]]

---
#### Mounting BitLocker-encrypted drives in Windows

The easiest method for mounting a BitLocker-encrypted virtual drive on Windows is to double-click the `.vhd` file. Since it is encrypted, Windows will initially show an error. After mounting, simply double-click the BitLocker volume to be prompted for the password.

![[Pasted image 20260404125813.png]]

#### Mounting BitLocker-encrypted drives in Linux (or macOS)

It is also possible to mount BitLocker-encrypted drives in Linux (or macOS). To do this, we can use a tool called [dislocker](https://github.com/Aorimn/dislocker). First, we need to install the package using `apt`:

```shell
sudo apt-get install dislocker
```
![[Pasted image 20260404133258.png]]

Next, we create two folders which we will use to mount the VHD.

```shell
sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount
```
![[Pasted image 20260404133332.png]]

We then use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device), decrypt the drive using `dislocker`, and finally mount the decrypted volume:

```shell
sudo losetup -f -P Backup.vhd

sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
# 1234qwer is the password
# Check if we have loop0p2 or loop0p1 or something else using lsblk

sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

![[Pasted image 20260404133449.png]]
![[Pasted image 20260404133521.png]]

If everything was done correctly, we can now browse the files:

```shell
cd /media/bitlockermount/
ls -la
```
![[Pasted image 20260404133549.png]]

Once we have analyzed the files on the mounted drive, we can unmount it using the following commands:

```shell
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```