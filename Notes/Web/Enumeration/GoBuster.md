
sudo apt install gobuster

| Flag | Long Flag     | Description                                                |
| ---- | ------------- | ---------------------------------------------------------- |
| -t   | --threads     | Number of concurrent threads (default 10)<br>keep maybe 64 |
| -v   | --verbose     | Verbose output                                             |
| -z   | --no-progress | Don't display progress                                     |
| -q   | --quiet       | Don't print the banner and other noise                     |
| -o   | --output      | Output file to write results to                            |
| -k   |               | pass the ssl sertificate error                             |

I will typically change the number of threads to 64 to increase the speed of my scans. If you don't change the number of threads, Gobuster can be a little slow.


Subdomain buster - 
gobuster vhost -u http://webenum.thm/Changes -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -k -t 64

---


Directory files buster-
gobuster dir -u http://webenum.thm/Changes -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -k -t 128 -x js,rb,pl,sh,py,so,gz,db,dbf,md,cn,cf,cc,pm,vb,go,ts,hs,cs,ml,html,aspx,conf,json,yaml,yml,text,init,info,lock,bash,java,data,logs,temp,dump,part,orig,back,save,oldf,dist -s 200,204,301,302,307,401,403 -b ""


2025/09/19 20:10:05 the server returns a status code that matches the provided options for non existing urls. https://kcecell.org/b2514cc7-5e6f-48d8-849d-ff7e49048b90 => 200 (Length: 461). Please exclude the response length or the status code or set the wildcard option.. To continue please exclude the status code or the length

curl -sk -I https://kcecell.org/$(uuidgen)
curl -sk https://kcecell.org/$(uuidgen) -o /dev/null -w '%{http_code} %{size_download}\n'

gobuster dir -u https://kcecell.org -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -k -t 64 --exclude-length 461,3681

---


Directory buster-
gobuster dir -u http://webenum.thm/Changes -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -k -t 64  




Cheat sheet - 
![[Pasted image 20250908231338.png]]