https://www.reddit.com/r/oscp/comments/1saijrp/what_not_to_study_for_the_oscp/

I took the OSCP after working as a pen tester for a little over 3 years. I wasted so much time studying things that were not on the exam. I went down every rabbit hole and ran out of time on my first try. Failed it!

It was really demoralizing as I worked in the field and did pen tests every day. Still the failure taught me what was covered on the exam.

I aced it the second time because I knew what NOT to study. You should know everything here. It just shouldn't be where you spend your time studying to pass the exam.

## OSINT

OSINT is important but the only OSINT skill you will need to pass the exam is how to find a public POC exploit.

You don't need to know anything about finding emails, subdomains, breached credentials, and so on. You should know how to use searchsploit and the google dork _site: github.com._

There just aren't that many practical ways to test OSINT in this exam.

## MITM Attacks

You won't need to relay NTLM but you should know how to pass the hash.

You won't need to know ARP poisoning, DNS spoofing, or need to use bettercap.

For the most part it just isn't practical to test people on this. You should know how to use Coercer and coercion attacks but not for the OSCP, probably.

## Advanced AD Attacks

No RBCD. No Golden Ticket. No trust abuse.

This is definitely an area that I went too deep on when studying for the OSCP and ended up in OSEP and CRTO territory without realizing it.

Know pass the hash and pass the ticket. Wouldn't hurt to know AD CS ESC1 but I don't think that's covered. It's good to know either way though as it's the top way that pen testers are getting domain admin these days.

## Web Attacks

Anything beyond basic knowledge is too much. You should know how SQLi works and how to do one. You should probably understand CSRF attacks as well. You should do a ton of CTF boxes and those simple web attacks that you do to get a foothold is what you will see.

You don't need to know cache attacks, CL TE attacks, XSS, probably not XXE and others.

## Web Shells

You _do_ need to know how to deploy a simple PHP webshell or similar. These are found in /usr/share/webshells on Kali. Test them out on CTF boxes. However, anything complex is overkill. Just really simple webshells.

## Exploit Dev

If you are doing anything besides changing the IP address in the exploit then you are doing too much and have gone down a rabbit hole. I often found myself trying to "fix" public POCs I had found only to find out it was the wrong exploit. There is no need for custom exploit development on the OSCP. If it doesn't pop from searchsploit or GitHub you probably don't need it.

## AV/EDR Evasion

NO AV or EDR on the OSCP machines! YAY. You can literally run mimikatz with no problem at all. Don't spend a moment worrying about this topic for the exam.

## Metasploit

Yes, you don't even need to use Metasploit once on the exam. There is a ton you could learn here but you are much better served doing it by hand. Need a reverse shell and forgot the syntax? Go to [revshells.com](http://revshells.com/?utm_source=www.credrelay.com&utm_medium=newsletter&utm_campaign=cred-relay-issue-3&_bhlid=478e09decdd68164f685b49ff7de91bd6ea4a3e7). Don't spend time on Metasploit for the exam.

I failed the OSCP because I studied the wrong things. Now you don't have to.

If you still want more, check out my OSCP prep checklist on my GitHub [here](https://github.com/jeffaf/oscp-prep-checklist?utm_source=www.credrelay.com&utm_medium=newsletter&utm_campaign=cred-relay-issue-3&_bhlid=3c3116b6cba45c032ea92602c1d86e536bcb4107).

I write about this and other stuff at my newsletter at [credrelay.com](https://www.credrelay.com/).