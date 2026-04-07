
Nearly all corporate environments include network shares used by employees to store and share files across teams.

They can unintentionally become a goldmine for attackers, especially when sensitive data like plaintext credentials or configuration files are left behind.

## Common credential patterns

- Look for keywords within files such as `passw`, `user`, `token`, `key`, and `secret`.
- Search for files with extensions commonly associated with stored credentials, such as `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`.
- Watch for files with "interesting" names that include terms like `config`, `user`, `passw`, `cred`, or `initial`.
- If you're trying to locate credentials within the `INLANEFREIGHT.LOCAL` domain, it may be helpful to search for files containing the string `INLANEFREIGHT\`.
- Keywords should be localized based on the target; if you are attacking a German company, it's more likely they will reference a `"Benutzer"` than a `"User"`.
- Pay attention to the shares you are looking at, and be strategic. If you scan ten shares with thousands of files each, it's going to take a significant amount of time. Shares used by `IT employees` might be a more valuable target than those used for company photos.

---
---
# Hunting in Windows

### Snaffler

[Snaffler](https://github.com/SnaffCon/Snaffler)

 When run on a **domain-joined** machine, automatically identifies accessible network shares and searches for interesting files. 
 
 The `README` file in the Github repository describes the numerous configuration options in great detail, however a basic search can be carried out like so:

```bash
Snaffler.exe -s
```

All of the tools covered in this section output a `large amount of information`. While they assist with automation, a fair amount of manual review is typically required, as many matches may turn out to be `"false positives"`. Two useful parameters that can help refine Snaffler's search process are:

- `-u` retrieves a list of users from Active Directory and searches for references to them in files
- `-i` and `-n` allow you to specify which shares should be included in the search

### PowerHuntShares

 [PowerHuntShares](https://github.com/NetSPI/PowerHuntShares)

A PowerShell script that doesn't necessarily need to be run on a domain-joined machine. 

One of its most useful features is that it generates an `HTML report` upon completion, providing an easy-to-use UI for reviewing the results:

```bash
PS C:\Users\Public\PowerHuntShares> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

How I downloaded it
My machine
![[Pasted image 20260404145117.png]]

Windows Machine
![[Pasted image 20260404145203.png]]

was not successful though



---
---

# Hunting from Linux

### MANSPIDER

If we don’t have access to a domain-joined computer, or simply prefer to search for files remotely, tools like [MANSPIDER](https://github.com/blacklanternsecurity/MANSPIDER) allow us to scan SMB shares from Linux. 

It's best to run `MANSPIDER` using the official Docker container to avoid dependency issues. 

Like the other tools, `MANSPIDER` offers many parameters that can be configured to fine-tune the search. A basic scan for files containing the string `passw` can be run as follows:

```bash
docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```


### NetExec

In addition to its many other uses, `NetExec` can also be used to search through network shares using the `--spider` option. 

This functionality is described in great detail on the [official wiki](https://www.netexec.wiki/smb-protocol/spidering-shares). A basic scan of network shares for files containing the string `"passw"` can be run like so:

```bash
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"
```

---
