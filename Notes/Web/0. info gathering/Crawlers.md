
letting a tool automatically visit every link it can find on a website and collect useful information for recon.

Think of a crawler as a **bot that clicks every link**, downloads every page, and extracts everything interesting:

- Emails
- Links
- Files
- JavaScript
- Hidden pages
- Comments
- Form fields
---
## Popular Web Crawlers

1. `Burp Suite Spider`
2. `OWASP ZAP (Zed Attack Proxy)`
3. `Scrapy (Python Framework)`: we will be using this
4. `Apache Nutch (Scalable Crawler)`: On Java, may be used for large scale

---
## Scrapy

### Installing Scrapy
```bash
pip3 install scrapy
```
![[Pasted image 20251208013433.png]]
### ReconSpider
```bash
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip

unzip ReconSpider.zip 
```
![[Pasted image 20251208013609.png]]
Run it
```bash
python3 ReconSpider.py http://inlanefreight.com
```
![[Pasted image 20251208013716.png]]
![[Pasted image 20251208013952.png]]


Result contains

|JSON Key|Description|
|---|---|
|`emails`|Lists email addresses found on the domain.|
|`links`|Lists URLs of links found within the domain.|
|`external_files`|Lists URLs of external files such as PDFs.|
|`js_files`|Lists URLs of JavaScript files used by the website.|
|`form_fields`|Lists form fields found on the domain (empty in this example).|
|`images`|Lists URLs of images found on the domain.|
|`videos`|Lists URLs of videos found on the domain (empty in this example).|
|`audio`|Lists URLs of audio files found on the domain (empty in this example).|
|`comments`|Lists HTML comments found in the source code.|


// not sure how to do recursion here