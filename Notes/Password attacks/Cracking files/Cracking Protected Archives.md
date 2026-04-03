
There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.

A comprehensive list of archive file types can be found on [FileInfo](https://fileinfo.com/filetypes/compressed). Rather than typing them out manually, we can also query the data using a one-liner, apply filters as needed, and save the results to a file.

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

