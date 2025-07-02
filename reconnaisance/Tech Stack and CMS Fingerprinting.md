# Technology Stack and CMS Fingerprinting
Technology Stack and CMS Fingerprinting of Live Hosts is a vital phase in reconnaissance during penetration testing and bug bounty hunting. It helps identify the underlying technologies and Content Management Systems (CMS) used by the target web application, which can reveal potential vulnerabilities and misconfigurations.

---

## Technology Stack Fingerprinting

### Using WhatWeb
It identifies websites' technologies including CMS, frameworks, web servers, etc.
  ```bash
  whatweb -v https://example.com
  ```
Whatweb also has a Web UI version. You can find it [here](https://www.whatweb.net/?trk=public_post-text).

### Wappalyzer (Browser Extension)
Download the [browser extension](https://chromewebstore.google.com/detail/wappalyzer-technology-pro/gppongmhjkpfnbhagpmjfkannfbllamg).
Visit any website and open the extension there. You will see all the technologies being used in the website like CMS, CDN, etc.

Wappalyzer also has a cli version which can be installed using npm.
  ```bash
  #Installation
  npm install -g wappalyzer
  #Usage
  wappalyzer https://example.com
  ```

### Using Nmap with NSE Scripts
  ```bash
  nmap -p80,443 --script http-enum,http-title,http-headers http://example.com
  ```

### Using other websites
Websites such as [BuiltWith](https://builtwith.com) and [Netcraft](https://sitereport.netcraft.com) to find the technologies used behind thhe target website.

## CMS Fingerprinting

### Using CMSeek
It is used in CMS detection and vulnerability scanning. It helps in detecting WordPress, Joomla, Drupal, etc., with plugin enumeration.
  ```bash
  #Installation
  git clone https://github.com/Tuhinshubhra/CMSeeK
  cd CMSeek
  pip install -r requirements.txt
  #Usage
  python3 cmseek.py
  ```

### Using WPScan (WordPress Specific)
WPScan helps in enumerating vulnerable plugins, themes, and users.
  ```bash
  wpscan --url https://example/ --api-token <API_KEY>
  ```
You need to get your API Key to run this tool after registering in the [WPScan Official website](https://wpscan.com/)

💡Pro Tip:
After getting the technologies used and their version, you can use [searchsploit](https://www.exploit-db.com/searchsploit), a CLI tool to get any possible exploits for them.