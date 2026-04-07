
# Enumeration

Drupal is yet another CMS tool. Quite widely used in the market.

Drupal is written in PHP and supports using MySQL or PostgreSQL for the backend. Additionally, SQLite can be used if there's no DBMS installed.

 Like WordPress, Drupal allows users to enhance their websites through the use of themes and modules.


## Discovery/Footprinting

A Drupal website can be identified in several ways, including by the header or footer message `Powered by Drupal`, the standard Drupal logo, the presence of a `CHANGELOG.txt` file or `README.txt file`, via the page source, or clues in the robots.txt file such as references to `/node`.

```shell
curl -s http://drupal.inlanefreight.local | grep Drupal
```

Another way to identify Drupal CMS is through [nodes](https://www.drupal.org/docs/8/core/modules/node/about-nodes). Drupal indexes its content using nodes. A node can hold anything such as a blog post, poll, article, etc. The page URIs are usually of the form `/node/<nodeid>`.

![[Pasted image 20260224210708.png]]

For example, the blog post above is found to be at `/node/1`. This representation is helpful in identifying a Drupal website when a custom theme is in use.

Drupal supports three types of users by default:

1. `Administrator`: This user has complete control over the Drupal website.
2. `Authenticated User`: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
3. `Anonymous`: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.

---
## Enumeration

Once we have discovered a Drupal instance, we can do a combination of manual and tool-based (automated) enumeration to uncover the version, installed plugins, and more. Depending on the Drupal version and any hardening measures that have been put in place, we may need to try several ways to identify the version number. Newer installs of Drupal by default block access to the `CHANGELOG.txt` and `README.txt` files, so we may need to do further enumeration. Let's look at an example of enumerating the version number using the `CHANGELOG.txt` file. To do so, we can use `cURL` along with `grep`, `sed`, `head`, etc.

  Drupal - Discovery & Enumeration

```shell
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```

![[Pasted image 20260224210857.png]]

There are several other things we could check in this instance to identify the version. Let's try a scan with `droopescan` as shown in the Joomla enumeration section. `Droopescan` has much more functionality for Drupal than it does for Joomla.

Let's run a scan against the `http://drupal.inlanefreight.local` host.

  Drupal - Discovery & Enumeration

```shell
droopescan scan drupal -u http://drupal.inlanefreight.local
```

Tried but was not able to install Droopescan in my computer. DAMNM.

so chat gpt asked me to use nuclei


---

A quick search for Drupal-related [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-1367/product_id-2387/Drupal-Drupal.html) will be helpful now. We would next want to look at installed plugins or abusing built-in functionality.

---
