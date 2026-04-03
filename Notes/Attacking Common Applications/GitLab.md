Its just like GitHub, it public company. Code is hosted on self owned server.

While looking at the repo,
- We may find scripts or configuration files that were accidentally committed containing cleartext secrets such as passwords that we may use to our advantage. 
- We may also come across SSH private keys. 
- We can attempt to use the search function to search for users, passwords, etc.

Applications such as GitLab allow for 
- public repositories (that require no authentication), 
- internal repositories (available to authenticated users), 
- and private repositories (restricted to specific users).

It is also worth reading 
- any public repositories for sensitive data 
- and, if the application allows, register an account and look to see if any interesting internal repositories are accessible.

A GitLab instance **can be** set up to allow anyone to register and then log in
OR
Require NO 2 factor authentication, If we can obtain user credentials from our OSINT, we may be able to log in to a GitLab instance

---
## Footprinting & Discovery

- We can quickly determine that GitLab is in use in an environment by just browsing to the GitLab URL, and we will be directed to the login page, which displays the GitLab logo.
- The only way to footprint the GitLab version number in use is by browsing to the `/help` page when logged in.
  ![[Pasted image 20260321161333.png]]
- If it allows anyone to register, then we can register one and check.

If we cannot register an account, we may have to try a low-risk exploit such as [this](https://www.exploit-db.com/exploits/49821). We do not recommend launching various exploits at an application, so if we have no way to enumerate the version number (such as a date on the page, the first public commit, or by registering a user), then we should stick to hunting for secrets and not try multiple exploits against it blindly. There have been a few serious exploits against GitLab [12.9.0](https://www.exploit-db.com/exploits/48431) and GitLab [11.4.7](https://www.exploit-db.com/exploits/49257) in the past few years as well as GitLab Community Edition [13.10.3](https://www.exploit-db.com/exploits/49821), [13.9.3](https://www.exploit-db.com/exploits/49944), and [13.10.2](https://www.exploit-db.com/exploits/49951).

---
## Enumeration

**There's not much we can do against GitLab without knowing the version number or being logged in**

1. The first thing we should try is browsing to `/explore` and see if there are any public projects that may contain something interesting.
2. Deep dive in Public repos to see if we can find find anything interesting
   - find production code that we can find a bug in after a code review, 
   - hard-coded credentials, 
   - a script or configuration file containing credentials, 
   - or other secrets such as an SSH private key or API key.
3. From here, we can explore each of the pages linked in the top left `groups`, `snippets`, and `help`
4. We can also use the search functionality and see if we can uncover any other projects.
5. Then we should try to register and sign in to see if we can access internal repos


We can also use the registration form to enumerate valid users, emails. 
If we can make a list of valid users, we could attempt to guess weak passwords or possibly re-use credentials that we find from a password dump using a tool such as `Dehashed`.

 Even if the `Sign-up enabled` checkbox is cleared within the settings page under `Sign-up restrictions`, we can still browse to the `/users/sign_up` page and enumerate users but will not be able to register a user.

![[Pasted image 20260321161122.png]]


Lets say we can access internal repos, we can check for creds, get the code and check for vulnerabilities or loopholes.

In a real-world scenario, we may be able to find a considerable amount of sensitive data if we can register and gain access to any of their repositories. As this [blog post](https://tillsongalloway.com/finding-sensitive-information-on-github/index.html) explains, there is a considerable amount of data that we may be able to uncover on GitLab, GitHub, etc.

---
---
# Attacking GitLab

## Username Enumeration

We can do Username Enumeration manually, of course, but scripts make our work much faster.
We can write one ourselves in Bash or Python or use [this one](https://www.exploit-db.com/exploits/49821) to enumerate a list of valid users. The Python3 version of this same tool can be found [here](https://github.com/dpgg101/GitLabUserEnum).

In versions below 16.6, GitLab's defaults are set to 10 failed login attempts, resulting in an automatic unlock after 10 minutes.
However, starting with GitLab version 16.6, administrators can now configure these values directly through the admin UI.
However, if these settings are not manually configured, they will still default to 10 failed login attempts and an unlock period of 10 minutes.

![[Pasted image 20260321171811.png]]
![[Pasted image 20260321171827.png]]

---

## Authenticated Remote Code Execution

As this is authenticated remote code execution, we first need a valid username and password.

GitLab Community Edition version 13.10.2 and lower suffered from an authenticated remote code execution [vulnerability](https://hackerone.com/reports/1154542) due to an issue with ExifTool handling metadata in uploaded image files.

We can use this [exploit](https://www.exploit-db.com/exploits/49951) to achieve RCE.

![[Pasted image 20260321174312.png]]
![[Pasted image 20260321174333.png]]

![[Pasted image 20260321174358.png]]

---
