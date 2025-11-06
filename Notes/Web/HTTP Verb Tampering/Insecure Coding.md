if a web page was found to be vulnerable to a SQL Injection vulnerability, and the back-end developer mitigated the SQL Injection vulnerability by the following applying input sanitization filters:

Code: php

```php
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}
```

We can see that the sanitization filter is only being tested on the `GET` parameter. If the GET requests do not contain any bad characters, then the query would be executed. However, when the query is executed, the `$_REQUEST["code"]` parameters are being used, which may also contain `POST` parameters, `leading to an inconsistency in the use of HTTP Verbs`.

In this case, an attacker may use a `POST` request to perform SQL injection, in which case the `GET` parameters would be empty (will not include any bad characters). The request would pass the security filter, which would make the function still vulnerable to SQL Injection.

---

if a security filter was being used to detect injection vulnerabilities and only checked for injections in `POST` parameters (e.g. `$_POST['parameter']`), it may be possible to bypass it by simply changing the request method to `GET`.

---

![[Pasted image 20251101220618.png]]
Fails
![[Pasted image 20251101220632.png]]

---

**NOTE: bur lets change it to post, but we simply cant change it to post ans in post the parameters are no passed via URL**
**so, right click > Change request method**
![[Pasted image 20251101223112.png]]
**so may be try both ways in url as well as body**

![[Pasted image 20251101222656.png]]

and this worked
![[Pasted image 20251101222756.png]]




