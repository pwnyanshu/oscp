
# Intro

- Open source ticketing tool
- PHP, MySQL
- compared to systems such as Jira, OTRS, Request Tracker, and Spiceworks

---
# Footprinting/Discovery/Enumeration

Looking back at our EyeWitness scan from earlier, we notice a screenshot of an osTicket instance which also shows that a cookie named `OSTSESSID` was set when visiting the page.
![[Pasted image 20260403192841.png]]

Also, most osTicket installs will showcase the osTicket logo with the phrase powered by in front of it in the page's footer. The footer may also contain the words Support Ticket System.
![[Pasted image 20260403192906.png]]

An Nmap scan will just show information about the webserver, such as Apache or IIS, and will not help us footprint the application.

We will not find many vulnerabilities and exploits that osTicket could have.

---

Even if the application is not vulnerable, it can still be used for our purposes. Here we can break down the main functions into the layers:

|`1. User input`|`2. Processing`|`3. Solution`|
|---|---|---|

#### User Input
For instance, from the osTicket [documentation](https://docs.osticket.com/en/latest/Getting%20Started/Post-Installation.html), we can see that only staff and users with administrator privileges can access the admin panel. So if our target company uses this or a similar application, we can cause a problem and "play dumb" and contact the company's staff. The simulated "lack of" knowledge about the services offered by the company in combination with a technical problem is a widespread social engineering approach to get more information from the company.

#### Solution

Depending on the depth of the problem, it is very likely that other staff members from the technical departments will be involved in the email correspondence. This will give us new email addresses to use against the osTicket admin panel (in the worst case) and potential usernames with which we can perform OSINT on or try to apply to other company services.


---

# Attacking osTicket

A search for osTicket on exploit-db shows various issues, including remote file inclusion, SQL injection, arbitrary file upload, XSS, etc. osTicket version 1.14.1 suffers from [CVE-2020-24881](https://nvd.nist.gov/vuln/detail/CVE-2020-24881) which was an SSRF vulnerability. If exploited, this type of flaw may be leveraged to gain access to internal resources or perform internal port scanning.

**Aside from web application-related vulnerabilities, support portals can sometimes be used to obtain an email address for a company domain, which can be used to sign up for other exposed applications requiring an email verification to be sent.**

Let's walk through a quick example, which is related to this [excellent blog post](https://medium.com/intigriti/how-i-hacked-hundreds-of-companies-through-their-helpdesk-b7680ddc2d4c)

If we come across a customer support portal during our assessment and can submit a new ticket, we may be able to obtain a valid company email address.

![[Pasted image 20260403194039.png]]

![[Pasted image 20260403194049.png]]

Now, if we log in, we can see information about the ticket and ways to post a reply. If the company set up their helpdesk software to correlate ticket numbers with emails, then any email sent to the email we received when registering, `940288@inlanefreight.local`, would show up here. 

With this setup, if we can find an external portal such as a Wiki, chat service (Slack, Mattermost, Rocket.chat), or a Git repository such as GitLab or Bitbucket, we may be able to use this email to register an account and the help desk support portal to receive a sign-up confirmation email.

---
# osTicket - Sensitive Data Exposure

Let's say we are on an external penetration test. During our OSINT and information gathering, we discover several user credentials using the tool [Dehashed](http://dehashed.com/)

Useful tool, used to find leaked passwords of a compony or so on.

```shell
sudo python3 dehashed.py -q inlanefreight.local -p
```

![[Pasted image 20260403194727.png]]

This dump shows cleartext passwords for two different users: `jclayton` and `kgrimes`. At this point, we have also performed subdomain enumeration and come across several interesting ones.

![[Pasted image 20260403194743.png]]

We browse to each subdomain and find that many are defunct, but some are promising.

![[Pasted image 20260403194847.png]]

Let's try the credentials for `jclayton`. No luck. We then try the credentials for `kgrimes` and have no success but noticing that the login page also accepts an email address, we try `kevin@inlanefreight.local` and get a successful login!

![[Pasted image 20260403194907.png]]

The user `kevin` appears to be a support agent but does not have any open tickets. Perhaps they are no longer active? In a busy enterprise, we would expect to see some open tickets. Digging around a bit, we find one closed ticket, a conversation between a remote employee and the support agent.

![[Pasted image 20260403195009.png]]

The employee states that they were locked out of their VPN account and asks the agent to reset it. The agent then tells the user that the password was reset to the standard new joiner password. The user does not have this password and asks the agent to call them to provide them with the password (solid security awareness!). The agent then commits an error and sends the password to the user directly via the portal. From here, we could try this password against the exposed VPN portal as the user may not have changed it.

Furthermore, the support agent states that this is the standard password given to new joiners and sets the user's password to this value. We have been in many organizations where the helpdesk uses a standard password for new users and password resets. Often the domain password policy is lax and does not force the user to change at the next login. If this is the case, it may work for other users. Though out of the scope of this module, in this scenario, it would be worth using tools like [linkedin2username](https://github.com/initstring/linkedin2username) to create a user list of company employees and attempt a password spraying attack against the VPN endpoint with this standard password.

Many applications such as osTicket also contain an address book. It would also be worth exporting all emails/usernames from the address book as part of our enumeration as they could also prove helpful in an attack such as password spraying.

---

- Limit what applications are exposed externally
- Enforce multi-factor authentication on all external portals
- Provide security awareness training to all employees and advise them not to use their corporate emails to sign up for third-party services
- Enforce a strong password policy in Active Directory and on all applications, disallowing common words such as variations of `welcome`, and `password`, the company name, and seasons and months
- Require a user to change their password after their initial login and periodically expire user's passwords

---

---

![[Pasted image 20260403184140.png]]![[Pasted image 20260403184320.png]]

Agent login
Just used the password fro, the lesson
![[Pasted image 20260403192229.png]]

![[Pasted image 20260403192617.png]]


![[Pasted image 20260403190935.png]]
