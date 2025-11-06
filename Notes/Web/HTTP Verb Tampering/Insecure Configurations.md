 A web server's authentication configuration may be limited to specific HTTP methods, which would leave some HTTP methods accessible without authentication. For example, a system admin may use the following configuration to require authentication on a particular web page:

Code: xml

```xml
<Limit GET POST>
    Require valid-user
</Limit>
```

As we can see, even though the configuration specifies both `GET` and `POST` requests for the authentication method, an attacker may still use a different HTTP method (like `HEAD`) to bypass this authentication mechanism altogether.

---

Lets try to reset the files

![[Pasted image 20251101212724.png]]

![[Pasted image 20251101212737.png]]

![[Pasted image 20251101212754.png]]

Failed as the web server is configured to compulsory authenticate for GET request


![[Pasted image 20251101215853.png]]
What all is actually allowed

---

![[Pasted image 20251101214056.png]]

Changing GET to PUT method

![[Pasted image 20251101214115.png]]

![[Pasted image 20251101214159.png]]

Yoo Success full, damn straight
