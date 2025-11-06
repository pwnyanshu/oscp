
### FoxyProxy

![[Pasted image 20251102183907.png]]

![[Pasted image 20251102183927.png]]

![[Pasted image 20251102183956.png]]

---

### Installoing CA certificate

![[Pasted image 20251102195852.png]]
got to https://burp

next cownload the CA certificate
![[Pasted image 20251102195942.png]]

next, come here and click view certificate
![[Pasted image 20251102200059.png]]

After that, we can select the `Authorities` tab, and then click on `import`, and select the downloaded CA certificate:

![Certificate Manager window showing Authorities tab with certificate list and options to View, Edit Trust, Import, Export, and Delete.](https://academy.hackthebox.com/storage/modules/110/firefox_import_cert.png)

Finally, we must select `Trust this CA to identify websites` and `Trust this CA to identify email users`, and then click OK: ![Downloading Certificate dialog asking to trust 'PortSwigger CA' for websites and email, with options to View, Examine, Cancel, and OK.](https://academy.hackthebox.com/storage/modules/110/firefox_trust_cert.jpg)

Once we install the certificate and configure the Firefox proxy, all Firefox web traffic will start routing through our web proxy.