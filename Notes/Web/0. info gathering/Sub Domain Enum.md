we will check DNS record (A or AAAA) to check if it resolved to an IP address
## WHY?
- `Development and Staging Environments`: Companies often use subdomains to test new features or updates before deploying them to the main site. Due to relaxed security measures, these environments sometimes contain vulnerabilities or expose sensitive information.
- `Hidden Login Portals`: Subdomains might host administrative panels or other login pages that are not meant to be publicly accessible. Attackers seeking unauthorised access can find these as attractive targets.
- `Legacy Applications`: Older, forgotten web applications might reside on subdomains, potentially containing outdated software with known vulnerabilities.
- `Sensitive Information`: Subdomains can inadvertently expose confidential documents, internal data, or configuration files that could be valuable to attackers.



### 1. Active Subdomain Enumeration
- info in DNS
- brute-force enumeration (`ffuf`,  `gobuster`)

### 2. Passive Subdomain Enumeration
- Certificate Transparency (CT) logs
- public repositories of SSL/TLS certificates (These certificates often include a list of associated subdomains in their Subject Alternative Name (SAN) field, providing a treasure trove of potential targets)
- **GOOGLE DORK**

---

# Subdomain Bruteforcing

