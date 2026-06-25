

**WHOIS:**

Commands:

whois inlanefreight.com

**DNS**

tools:

**dig**

Commands:

dig domain.com
dig domain.com A
dig domain.com AAAA
dig domain.com MX
dig domain.com NS
dig domain.com TXT
dig domain.com CNAME
dig domain.com SOA
dig @1.1.1.1 domain.com
dig +trace domain.com
dig -x 192.168.1.1
dig +short domain.com - Provides a short, concise answer to the query.
dig +noall +answer domain.com -Displays only the answer section of the query output.
dig domain.com ANY


**nslookup**
**host**
**dnsenum**
**fierce**
**dnsrecon**
**theHarvester**
**Online DNS Lookup Services**

**Subdomains**

Active:

**dnsenum**

Commands:

dnsenum --enum inlanefreight.com -f /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt  -r

dnsenum --enum inlanefreight.com -f  /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt 


**gobuster**
**ffuf**

**DNS ZONE TRANFERS**

dig axfr @nsztm1.digi.ninja zonetransfer.me

dig axfr @nsztm1.digi.ninja zonetransfer.me

Passive:
**Certificate Transparency (CT) logs**

[crt.sh](https://crt.sh/)
[Censys](https://search.censys.io/)

**Search engines**

**FINGERPRINTING**

| Tool         | Description                                                                                                           | Features                                                                                            |
| ------------ | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `Wappalyzer` | Browser extension and online service for website technology profiling.                                                | Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more. |
| `BuiltWith`  | Web technology profiler that provides detailed reports on a website's technology stack.                               | Offers both free and paid plans with varying levels of detail.                                      |
| `WhatWeb`    | Command-line tool for website fingerprinting.                                                                         | Uses a vast database of signatures to identify various web technologies.                            |
| `Nmap`       | Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting. | Can be used with scripts (NSE) to perform more specialised fingerprinting.                          |
| `Netcraft`   | Offers a range of web security services, including website fingerprinting and security reporting.                     | Provides detailed reports on a website's technology, hosting provider, and security posture.        |
| `wafw00f`    | Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).                             | Helps determine if a WAF is present and, if so, its type and configuration.                         |

```
curl -I inlanefreight.com
```

If firewall present: 

```
wafw00f inlanefreight.com
```

**Nikto**

```
oib3u@htb[/htb]$ sudo apt update && sudo apt install -y perl
oib3u@htb[/htb]$ git clone https://github.com/sullo/nikto
oib3u@htb[/htb]$ cd nikto/program
oib3u@htb[/htb]$ chmod +x ./nikto.pl
```


```
oib3u@htb[/htb]$ nikto -h inlanefreight.com -Tuning b
```



**CRAWLING**

robots.txt

| Directive     | Description                                                                                                        | Example                                                      |
| ------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| `Disallow`    | Specifies paths or patterns that the bot should not crawl.                                                         | `Disallow: /admin/` (disallow access to the admin directory) |
| `Allow`       | Explicitly permits the bot to crawl specific paths or patterns, even if they fall under a broader `Disallow` rule. | `Allow: /public/` (allow access to the public directory)     |
| `Crawl-delay` | Sets a delay (in seconds) between successive requests from the bot to avoid overloading the server.                | `Crawl-delay: 10` (10-second delay between requests)         |
| `Sitemap`     | Provides the URL to an XML sitemap for more efficient crawling.                                                    | `Sitemap: https://www.example.com/sitemap.xml`               |

**WELL-KNOWN URIs**

| URI Suffix                     | Description                                                                                           | Status      | Reference                                                                                                                                                                          |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `security.txt`                 | Contains contact information for security researchers to report vulnerabilities.                      | Permanent   | RFC 9116                                                                                                                                                                           |
| `/.well-known/change-password` | Provides a standard URL for directing users to a password change page.                                | Provisional | [https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri](https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri) |
| `openid-configuration`         | Defines configuration details for OpenID Connect, an identity layer on top of the OAuth 2.0 protocol. | Permanent   | [http://openid.net/specs/openid-connect-discovery-1_0.html](http://openid.net/specs/openid-connect-discovery-1_0.html)                                                             |
| `assetlinks.json`              | Used for verifying ownership of digital assets (e.g., apps) associated with a domain.                 | Permanent   | [https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md](https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md)         |
| `mta-sts.txt`                  | Specifies the policy for SMTP MTA Strict Transport Security (MTA-STS) to enhance email security.      | Permanent   | RFC 8461                                                                                                                                                                           |


**CREEPY CRAWLIES**

**Burp Suite Spider**
**OWASP ZAP**
**Scrapy**
**Apache Nutch**
**ReconSpider**

```
oib3u@htb[/htb]$ wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
oib3u@htb[/htb]$ unzip ReconSpider.zip
```

```
python3 ReconSpider.py http://inlanefreight.com
```

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
**Search Engine Discovery**

| Operator                | Operator Description                                         | Example                                             | Example Description                                                                     |
| :---------------------- | :----------------------------------------------------------- | :-------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| `site:`                 | Limits results to a specific website or domain.              | `site:example.com`                                  | Find all publicly accessible pages on example.com.                                      |
| `inurl:`                | Finds pages with a specific term in the URL.                 | `inurl:login`                                       | Search for login pages on any website.                                                  |
| `filetype:`             | Searches for files of a particular type.                     | `filetype:pdf`                                      | Find downloadable PDF documents.                                                        |
| `intitle:`              | Finds pages with a specific term in the title.               | `intitle:"confidential report"`                     | Look for documents titled "confidential report" or similar variations.                  |
| `intext:` or `inbody:`  | Searches for a term within the body text of pages.           | `intext:"password reset"`                           | Identify webpages containing the term “password reset”.                                 |
| `cache:`                | Displays the cached version of a webpage (if available).     | `cache:example.com`                                 | View the cached version of example.com to see its previous content.                     |
| `link:`                 | Finds pages that link to a specific webpage.                 | `link:example.com`                                  | Identify websites linking to example.com.                                               |
| `related:`              | Finds websites related to a specific webpage.                | `related:example.com`                               | Discover websites similar to example.com.                                               |
| `info:`                 | Provides a summary of information about a webpage.           | `info:example.com`                                  | Get basic details about example.com, such as its title and description.                 |
| `define:`               | Provides definitions of a word or phrase.                    | `define:phishing`                                   | Get a definition of "phishing" from various sources.                                    |
| `numrange:`             | Searches for numbers within a specific range.                | `site:example.com numrange:1000-2000`               | Find pages on example.com containing numbers between 1000 and 2000.                     |
| `allintext:`            | Finds pages containing all specified words in the body text. | `allintext:admin password reset`                    | Search for pages containing both "admin" and "password reset" in the body text.         |
| `allinurl:`             | Finds pages containing all specified words in the URL.       | `allinurl:admin panel`                              | Look for pages with "admin" and "panel" in the URL.                                     |
| `allintitle:`           | Finds pages containing all specified words in the title.     | `allintitle:confidential report 2023`               | Search for pages with "confidential," "report," and "2023" in the title.                |
| `AND`                   | Narrows results by requiring all terms to be present.        | `site:example.com AND (inurl:admin OR inurl:login)` | Find admin or login pages specifically on example.com.                                  |
| `OR`                    | Broadens results by including pages with any of the terms.   | `"linux" OR "ubuntu" OR "debian"`                   | Search for webpages mentioning Linux, Ubuntu, or Debian.                                |
| `NOT`                   | Excludes results containing the specified term.              | `site:bank.com NOT inurl:login`                     | Find pages on bank.com excluding login pages.                                           |
| `*` (wildcard)          | Represents any character or word.                            | `site:socialnetwork.com filetype:pdf user* manual`  | Search for user manuals (user guide, user handbook) in PDF format on socialnetwork.com. |
| `..` (range search)     | Finds results within a specified numerical range.            | `site:ecommerce.com "price" 100..500`               | Look for products priced between 100 and 500 on an e-commerce website.                  |
| `" "` (quotation marks) | Searches for exact phrases.                                  | `"information security policy"`                     | Find documents mentioning the exact phrase "information security policy".               |
| `-` (minus sign)        | Excludes terms from the search results.                      | `site:news.com -inurl:sports`                       | Search for news articles on news.com excluding sports-related content.                  |

**Web Archives**

waybackmachine

**Automating Recon**

[FinalRecon](https://github.com/thewhiteh4t/FinalRecon)


git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help


```
oib3u@htb[/htb]$ ./finalrecon.py --headers --whois --url http://inlanefreight.com
```


[Recon-ng](https://github.com/lanmaster53/recon-ng)
[theHarvester](https://github.com/laramies/theHarvester)
[SpiderFoot](https://github.com/smicallef/spiderfoot)
[OSINT Framework](https://osintframework.com/)
