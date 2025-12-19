
### **What it is**

`.well-known` (RFC 8615) is a **standardized directory** under a domain (`/.well-known/`) that stores **metadata, configuration files, and security-related info**.

Purpose:  
Provide a **consistent, machine-discoverable location** for important service/configuration endpoints.

Example:

`https://example.com/.well-known/security.txt`

---

### **Why it matters**

- Makes discovery of config + policy files **automatic** for browsers, apps, and security tools.
- Helps in **web recon**, revealing authentication configs, security contacts, asset verifications, etc.

---

### **Common .well-known URIs**

|URI|Purpose|
|---|---|
|`security.txt`|Contact info for reporting vulnerabilities (RFC 9116).|
|`change-password`|Standard URL for password-change pages.|
|`openid-configuration`|OpenID Connect provider metadata (OAuth 2.0).|
|`assetlinks.json`|Verifies ownership of digital assets/apps.|
|`mta-sts.txt`|SMTP Strict Transport Security policy.|

security.txt, this has some hiring info too which is helpful
![[Pasted image 20251208002336.png]]



---

### **OpenID Configuration Example**

Access at:

`https://example.com/.well-known/openid-configuration`

Reveals JSON metadata like:

- Authorization endpoint
- Token endpoint
- Userinfo endpoint
- JWKS URI
- Supported scopes, algorithms, response types

Useful for mapping authentication flow, cryptographic keys, and identity structure.

---

### **Recon Use Cases**

- Discover hidden service endpoints
- Enumerate auth-related configurations
- Identify misconfigurations in OAuth / OpenID setups
- Gather security contact info
- Validate domain–application relationships
